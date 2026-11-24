# System Design and FastAPI Basics (AI / RAG)

> The example used everywhere is a **RAG chatbot**: an AI that answers questions using company documents (PDFs).
> In Part 3 we take this chatbot from **1 server** to **1 million users**.

---

## Table of Contents

- [Part 1: System Design Basics](#part-1-system-design-basics)
  - [1.1 What is System Design](#11-what-is-system-design)
  - [1.2 Words You Must Know](#12-words-you-must-know)
  - [1.3 Vertical vs Horizontal Scaling](#13-vertical-vs-horizontal-scaling)
  - [1.4 Load Balancer](#14-load-balancer)
  - [1.5 Stateless Servers](#15-stateless-servers)
  - [1.6 Caching](#16-caching)
  - [1.7 Databases](#17-databases)
  - [1.8 Queues and Background Workers](#18-queues-and-background-workers)
  - [1.9 Rate Limiting](#19-rate-limiting)
  - [1.10 Timeouts, Retries and Fallbacks](#110-timeouts-retries-and-fallbacks)
  - [1.11 Monitoring](#111-monitoring)
  - [1.12 CAP Theorem in One Minute](#112-cap-theorem-in-one-minute)
- [Part 2: FastAPI Basics](#part-2-fastapi-basics)
  - [2.1 What is FastAPI and Why Use It](#21-what-is-fastapi-and-why-use-it)
  - [2.2 Your First API](#22-your-first-api)
  - [2.3 Path and Query Parameters](#23-path-and-query-parameters)
  - [2.4 Request Body with Pydantic](#24-request-body-with-pydantic)
  - [2.5 Response Model](#25-response-model)
  - [2.6 async def vs def](#26-async-def-vs-def)
  - [2.7 Dependency Injection](#27-dependency-injection)
  - [2.8 Errors with HTTPException](#28-errors-with-httpexception)
  - [2.9 Lifespan: Create Things Once at Startup](#29-lifespan-create-things-once-at-startup)
  - [2.10 Background Tasks](#210-background-tasks)
  - [2.11 Streaming the LLM Answer](#211-streaming-the-llm-answer)
  - [2.12 Middleware](#212-middleware)
  - [2.13 File Upload](#213-file-upload)
  - [2.14 Testing](#214-testing)
  - [2.15 Running in Production](#215-running-in-production)
- [Part 3: Scenario: Scale a RAG System to 1M Users](#part-3-scenario-scale-a-rag-system-to-1m-users)
  - [3.1 What We Are Building](#31-what-we-are-building)
  - [3.2 Step 1: Ask Questions First](#32-step-1-ask-questions-first)
  - [3.3 Step 2: Do the Math](#33-step-2-do-the-math)
  - [3.4 Step 3: Version 1, One Server](#34-step-3-version-1-one-server)
  - [3.5 Step 4: What Breaks When Users Grow](#35-step-4-what-breaks-when-users-grow)
  - [3.6 Step 5: Final Design for 1M Users](#36-step-5-final-design-for-1m-users)
  - [3.7 One Request, Step by Step](#37-one-request-step-by-step)
  - [3.8 How Each Part Scales](#38-how-each-part-scales)
  - [3.9 Code: A Scalable RAG API](#39-code-a-scalable-rag-api)
  - [3.10 Saving Money](#310-saving-money)
  - [3.11 Bottlenecks and Fixes](#311-bottlenecks-and-fixes)
  - [3.12 How to Answer This in the Interview](#312-how-to-answer-this-in-the-interview)
- [Part 4: Common Interview Questions](#part-4-common-interview-questions)
- [Part 5: One-Page Cheat Sheet](#part-5-one-page-cheat-sheet)

---

# Part 1: System Design Basics

## 1.1 What is System Design

**System design** means planning how the parts of a software system work together, so that it:

- is **fast**,
- **does not go down**,
- can **grow** when more users come.

The simplest system looks like this:

```
   User ──► API Server ──► Database
```

Everything in this file is about one question: **how do we make this simple picture work for many more users?**

---

## 1.2 Words You Must Know

| Word | Simple meaning | RAG example |
|---|---|---|
| **Latency** | Time taken by one request | User waits 3 seconds for an answer |
| **Throughput** | How many requests the system handles per second | 200 questions per second |
| **QPS / RPS** | Queries (or Requests) Per Second | "Peak load is 200 QPS" |
| **p50 / p95 / p99** | p95 = 95% of requests are faster than this number | p95 = 4 s means only 5% of users wait more than 4 s |
| **Concurrency** | Requests in progress at the same moment | 1,000 users waiting for the LLM at the same time |
| **Availability** | % of time the system works | 99.9% = about 43 minutes down per month |
| **Scalability** | Handle more users by adding more machines | Add more API servers when traffic grows |
| **Bottleneck** | The slowest part, which limits everything else | The LLM call |
| **SLA** | A promise to the customer about uptime and speed | "99.9% uptime, or we refund you" |
| **Token** | A piece of a word. LLM cost and limits are counted in tokens | 1 page of text ≈ 500 to 700 tokens |
| **TPM** | Tokens Per Minute: the limit your LLM provider gives you | "Our deployment allows X tokens per minute" |
| **TTFT** | Time To First Token: time until the user sees the first word | 0.8 s |

**Why p95 and not average?** The average hides the slow users. If 95 users wait 1 s and 5 users wait 60 s, the average looks fine (about 4 s), but 5 users are very unhappy. p95 and p99 show this pain.

---

## 1.3 Vertical vs Horizontal Scaling

Think of a restaurant with too many customers.

- **Vertical scaling (scale up)** = make the one chef faster. In computers: buy a **bigger machine** (more CPU, more RAM).
- **Horizontal scaling (scale out)** = hire **more chefs**. In computers: add **more machines** and share the work.

| | Vertical (bigger machine) | Horizontal (more machines) |
|---|---|---|
| How | Upgrade CPU / RAM / GPU | Add more servers behind a load balancer |
| Good | Very simple, no code change | Almost no limit, if one dies others keep working |
| Bad | There is a maximum size. One machine = one point of failure | Needs a load balancer, and servers must be **stateless** |
| RAG use | A bigger GPU for a self-hosted model | More FastAPI servers for 1M users |

**Rule for interviews:** for many users, say **horizontal scaling** for the API servers.

---

## 1.4 Load Balancer

A **load balancer** sits in front of your servers and sends each request to one healthy server.
Like a restaurant host who takes each guest to a free table.

```
                             ┌──► Server 1  (healthy)
   Users ──► Load Balancer ──┼──► Server 2  (healthy)
                             └──► Server 3  (sick: gets no traffic)
```

How it picks a server:

- **Round robin**: 1, 2, 3, 1, 2, 3, ... Simple.
- **Least connections**: send to the server with the fewest open requests.
  This is **better for LLM apps**, because some requests take 1 second and some take 20 seconds.

**Health checks:** the load balancer calls something like `GET /health` every few seconds.
If a server does not answer, it stops sending traffic there.

Examples: Nginx, AWS ALB, Azure Application Gateway, Azure Front Door.

---

## 1.5 Stateless Servers

A **stateless** server does **not** keep user data in its own memory between requests.
All data lives in a shared place (Redis or a database).

Why this matters:

- Any server can handle any request, so the load balancer can send a user anywhere.
- You can add or remove servers at any time.
- If a server crashes, no data is lost.

```python
# BAD: chat history is stored in this server's memory.
# Problem 1: the user's next request may go to a DIFFERENT server, which has no history.
# Problem 2: if this server restarts, all history is gone.
chat_history = {}

def save_message(session_id: str, message: str):
    chat_history.setdefault(session_id, []).append(message)


# GOOD: chat history is stored in Redis, which ALL servers share.
import redis

r = redis.Redis(host="localhost", port=6379, decode_responses=True)

def save_message(session_id: str, message: str):
    key = f"chat:{session_id}"
    r.rpush(key, message)        # add the message to the end of a list
    r.expire(key, 60 * 60 * 24)  # delete the history automatically after 24 hours
```

---

## 1.6 Caching

A **cache** stores results you already computed, so the next time is fast and cheap.
Like a restaurant keeping popular dishes ready, instead of cooking each one from zero.

Words to know:

- **Cache hit**: the answer was in the cache. Fast.
- **Cache miss**: not in the cache. Do the slow work, then save the result.
- **TTL (Time To Live)**: how long an item stays in the cache before it is deleted.
- **Eviction (LRU)**: when the cache is full, delete the **Least Recently Used** items first.

**Cache-aside pattern** (the most common one):

1. Look in the cache.
2. Found? Return it.
3. Not found? Do the real work (call the LLM), save the result in the cache, then return it.

```python
import hashlib
import json

def get_answer(question: str) -> str:
    # Make a short, fixed-length key from the question text.
    key = "answer:" + hashlib.sha256(question.lower().encode()).hexdigest()

    cached = r.get(key)                # 1. look in the cache
    if cached:
        return json.loads(cached)      # 2. cache hit: about 1 ms and free

    answer = run_rag_pipeline(question)        # 3. cache miss: slow (seconds) and costs money
    r.set(key, json.dumps(answer), ex=3600)    #    save it for 1 hour (TTL)
    return answer
```

**Three caches in a RAG system:**

| Cache | What it stores | Saves |
|---|---|---|
| **Answer cache** | Exact same question → saved answer | The whole pipeline, including the LLM call |
| **Semantic cache** | Question with the *same meaning* → saved answer | LLM calls for reworded questions |
| **Embedding cache** | Text → its vector | Embedding API calls |

**The hard part of caching is stale data.** If a document changes, old cached answers become wrong.
Fix: use a TTL, and clear the cache when documents are updated.

---

## 1.7 Databases

| Type | Good for | Examples |
|---|---|---|
| **SQL (relational)** | Structured data, relationships, transactions | PostgreSQL, MySQL, Azure SQL |
| **NoSQL (document / key-value)** | Flexible data, very high write volume, easy to spread across machines | MongoDB, Cosmos DB, DynamoDB |
| **Key-value in memory** | Cache, sessions, counters | Redis |
| **Vector DB** | Finding similar text by meaning (embeddings) | Azure AI Search, pgvector, Qdrant, Pinecone |
| **Blob / object storage** | Big files | Azure Blob Storage, AWS S3 |

**Where does RAG data go?**

| Data | Store |
|---|---|
| Users, companies, billing | PostgreSQL (SQL) |
| Chat history | Cosmos DB / MongoDB / PostgreSQL |
| Chunks + vectors | Vector DB |
| Original PDFs | Blob storage |
| Cache, rate-limit counters | Redis |

**How to scale a database:**

- **Index**: like the index at the back of a book. Find rows without reading every row.
- **Read replicas**: copies of the database that only serve reads. Most apps read much more than they write.
- **Sharding (partitioning)**: split the data across many machines by a key, for example `tenant_id` or `user_id`.
- **Connection pooling**: reuse open connections instead of opening a new one per request.

---

## 1.8 Queues and Background Workers

Some work is slow. For example, processing a 300-page PDF can take minutes.
**Do not make the user wait.** Put a "job" in a **queue**, reply "we got it", and let **workers** do the job in the background.

Like a restaurant: the waiter writes an order ticket and moves on. The kitchen picks up tickets one by one.

```
   Upload API ──► Queue [job][job][job] ──► Worker 1
                                        ──► Worker 2
                                        ──► Worker 3
```

Why queues are great:

- The API stays fast.
- **Spikes are absorbed**: if 10,000 PDFs arrive at once, they wait in the queue instead of crashing the server.
- Failed jobs can be **retried**.
- You can scale workers separately from API servers.

Tools: Redis + Celery or RQ, RabbitMQ, Kafka, AWS SQS, Azure Service Bus.

**In RAG:** document ingestion (parse → chunk → embed → save to vector DB) always goes through a queue.

---

## 1.9 Rate Limiting

**Rate limiting** = limit how many requests one user can send in a time window.
Example: 20 questions per minute per user.

Why:

- Protects your servers from abuse and bots.
- Protects your **LLM bill** and your LLM quota.
- Keeps the system fair for everyone.

If the user is over the limit, return **HTTP 429 Too Many Requests**.

Two common methods:

- **Fixed window**: count requests in each minute. Simple. (Code in [Part 3](#39-code-a-scalable-rag-api).)
- **Token bucket**: a bucket fills with "tokens" at a steady speed. Each request uses one. Allows small bursts.

Keep the counter in **Redis**, so the limit works across **all** servers, not just one.

---

## 1.10 Timeouts, Retries and Fallbacks

Outside services (LLM API, vector DB) will sometimes be slow or fail. Plan for it.

- **Timeout**: never wait forever. Example: give up on the LLM after 30 s.
- **Retry with exponential backoff**: wait 1 s, then 2 s, then 4 s... and add a small random delay (**jitter**), so thousands of clients do not all retry at the same moment.
- **Only retry errors that can get better**: 429 (too many requests), 500, 503, timeouts. **Do not retry 400** (bad request). It will fail again.
- **Fallback**: if the main model fails, use a backup model or a backup region.
- **Circuit breaker**: if a service keeps failing, stop calling it for a short time. Like a fuse in your house. This gives it time to recover.
- **Graceful degradation**: give a smaller answer instead of an error. Example: if the LLM is down, show the top matching document passages.

```python
import asyncio
import random

async def call_with_retry(func, max_tries: int = 3):
    for attempt in range(max_tries):
        try:
            return await func()
        except TimeoutError:
            if attempt == max_tries - 1:
                raise                              # last try failed: give up
            wait = (2 ** attempt) + random.random()  # 1.x s, 2.x s, 4.x s ...
            await asyncio.sleep(wait)
```

> Tip: the official OpenAI Python SDK already retries 429 / 5xx errors with backoff.
> You set it with `AsyncOpenAI(max_retries=3, timeout=30)`.

---

## 1.11 Monitoring

You cannot fix what you cannot see. The three pillars:

- **Logs**: text records of what happened. "User 42 asked X, error Y."
- **Metrics**: numbers over time. "p95 latency = 3.2 s, error rate = 0.5%."
- **Traces**: the path of **one** request through all services, with the time spent in each step.

**What to watch in a RAG system:**

- p95 latency and TTFT
- Error rate (especially LLM 429 errors)
- Tokens per request and **cost per day**
- Cache hit rate
- Answer quality (faithfulness, relevance; see the Ragas section in [RAG.md](RAG.md))

Set **alerts**, for example: "page the on-call person if the error rate is above 2% for 5 minutes."

Tools: Prometheus + Grafana, Azure Monitor / Application Insights, OpenTelemetry, Langfuse or LangSmith (for LLM traces).

---

## 1.12 CAP Theorem in One Minute

When your data is on many machines and the **network between them breaks** (a **P**artition), you must choose one:

- **C**onsistency: everyone sees the same, latest data. The system may refuse to answer until it is sure.
- **A**vailability: the system always answers, but the data may be a little old.

Examples:

- **Bank balance** → choose **consistency**. A wrong balance is not OK.
- **RAG chatbot** → choose **availability**. If a newly uploaded PDF shows up in search 1 minute later, that is fine.
  This is called **eventual consistency**: the data becomes correct after a short time.

---

# Part 2: FastAPI Basics

## 2.1 What is FastAPI and Why Use It

**FastAPI** is a Python framework for building web APIs.

Why it is popular for AI apps:

- **Fast**: built on async Python (Starlette). It can handle many waiting requests at once.
- **Automatic validation**: you describe the input with Python types (Pydantic), and bad input is rejected for you.
- **Automatic docs**: open `/docs` in the browser and you get an interactive Swagger page.
- **Python**: the same language as your ML and LLM code.

Install:

```bash
pip install "fastapi[standard]"   # FastAPI + uvicorn server + file upload support
```

---

## 2.2 Your First API

```python
# main.py
from fastapi import FastAPI

# Create the app. This object holds all our routes (URLs).
app = FastAPI(title="RAG Chatbot API")


# When someone sends "GET /health", FastAPI runs this function.
@app.get("/health")
def health():
    # FastAPI turns this Python dict into JSON for us.
    return {"status": "ok"}
```

Run it:

```bash
uvicorn main:app --reload     # main = file name, app = the FastAPI object
# or
fastapi dev main.py
```

Open `http://127.0.0.1:8000/docs` to see and test your API in the browser.

**HTTP methods:**

| Method | Use | RAG example |
|---|---|---|
| GET | Read data | Get a document's details |
| POST | Send / create | Ask a question, upload a PDF |
| PUT | Replace | Replace a document |
| PATCH | Change part of something | Rename a document |
| DELETE | Remove | Delete a document |

**Status codes to remember:**

| Code | Meaning |
|---|---|
| 200 | OK |
| 201 | Created |
| 202 | Accepted (we will process it later; used with queues) |
| 400 | Bad request (your input is wrong) |
| 401 | Not logged in |
| 403 | Logged in, but not allowed |
| 404 | Not found |
| 422 | Validation error (FastAPI returns this when input does not match your model) |
| 429 | Too many requests (rate limit) |
| 500 | Server error (our bug) |
| 503 | Service unavailable (overloaded, or a dependency like the LLM is down) |

---

## 2.3 Path and Query Parameters

```python
@app.get("/documents/{doc_id}")
def get_document(doc_id: int, include_chunks: bool = False):
    # doc_id is a PATH parameter: it is part of the URL.
    #   /documents/42
    # include_chunks is a QUERY parameter: it comes after "?".
    #   /documents/42?include_chunks=true
    #
    # FastAPI converts "42" (text) to 42 (int) for us.
    # If the user sends /documents/abc, FastAPI returns a 422 error automatically.
    return {"doc_id": doc_id, "include_chunks": include_chunks}
```

---

## 2.4 Request Body with Pydantic

For POST requests, the data comes in the **body** as JSON. We describe its shape with a **Pydantic model**.

```python
from pydantic import BaseModel, Field


class AskRequest(BaseModel):
    # "..." means required. min/max length protect us from empty or huge inputs.
    # max_length also protects the LLM bill: a 1 MB "question" costs a lot of tokens.
    question: str = Field(..., min_length=3, max_length=2000)

    # Optional field with a default value, and a safe range (1 to 20).
    top_k: int = Field(5, ge=1, le=20)

    # Optional: can be a string or missing (None).
    session_id: str | None = None


@app.post("/ask")
def ask(req: AskRequest):
    # By the time we are here, the input is already checked.
    # If "question" was missing or too short, FastAPI already returned 422.
    return {"you_asked": req.question, "top_k": req.top_k}
```

Example request body:

```json
{ "question": "What is our refund policy?", "top_k": 5 }
```

---

## 2.5 Response Model

A **response model** describes what the API sends back.

```python
class Source(BaseModel):
    doc_name: str
    page: int


class AskResponse(BaseModel):
    answer: str
    sources: list[Source]


@app.post("/ask", response_model=AskResponse)
def ask(req: AskRequest):
    return {
        "answer": "Refunds are allowed within 30 days.",
        "sources": [{"doc_name": "policy.pdf", "page": 4}],
        "internal_score": 0.91,   # NOT in AskResponse, so FastAPI removes it from the output
    }
```

Why use it:

- The output always has the same shape. The frontend team can trust it.
- It shows up in `/docs`.
- It **hides internal fields** you did not mean to send.

---

## 2.6 async def vs def

**This is the most asked FastAPI question.**

Think of a waiter:

- **Sync waiter**: takes an order, then **stands at the kitchen and waits** until the dish is ready. Other tables wait.
- **Async waiter**: takes an order, gives it to the kitchen, and **serves other tables while the dish is cooking**.

A RAG app spends most of its time **waiting**: for the LLM API, the vector DB, Redis.
This is called **I/O-bound** work. Async is perfect for it. One server process can handle hundreds of waiting requests.

**Rules:**

| Situation | Use |
|---|---|
| Calling async libraries (`AsyncOpenAI`, `httpx.AsyncClient`, `redis.asyncio`) | `async def` + `await` |
| Calling blocking libraries (`requests`, old DB drivers) | plain `def` (FastAPI runs it in a thread pool for you) |
| Heavy CPU work (running a local model, parsing big PDFs) | Separate worker process, not inside the API |
| `time.sleep()` or `requests.get()` inside `async def` | **Never.** It freezes the server for **all** users |

```python
import asyncio
import time


# BAD: blocking call inside "async def".
# For 3 seconds, this server process cannot serve ANY other user.
@app.get("/bad")
async def bad():
    time.sleep(3)
    return {"msg": "done"}


# GOOD: "await" says "I am waiting. Go help other requests meanwhile."
@app.get("/good")
async def good():
    await asyncio.sleep(3)
    return {"msg": "done"}
```

**Do two things at the same time with `asyncio.gather`:**

```python
@app.post("/ask")
async def ask(req: AskRequest):
    # Search the vector DB AND load chat history at the same time.
    # Total time = the slower of the two, not the sum.
    chunks, history = await asyncio.gather(
        search_chunks(req.question),
        load_chat_history(req.session_id),
    )
    ...
```

---

## 2.7 Dependency Injection

`Depends` tells FastAPI: "**before** running my endpoint, run this helper function and give me its result."

Common uses: login check, database session, rate limiting, settings.

```python
from fastapi import Depends, Header, HTTPException

# DEMO ONLY. In real life, check a JWT token or look up the key in a database.
API_KEYS = {"key-123": "user_1", "key-456": "user_2"}


def get_current_user(x_api_key: str = Header(...)):
    # Header(...) reads the "x-api-key" header from the request.
    # (FastAPI turns the underscore in x_api_key into a dash for the header name.)
    user = API_KEYS.get(x_api_key)
    if user is None:
        raise HTTPException(status_code=401, detail="Invalid API key")
    return user


@app.get("/me")
def me(user: str = Depends(get_current_user)):
    # FastAPI already ran get_current_user. If the key was bad, we never get here.
    return {"user": user}
```

Why it is useful:

- Write the login check **once**, reuse it on every endpoint.
- In tests, swap the real dependency for a fake one:
  `app.dependency_overrides[get_current_user] = lambda: "test_user"`

---

## 2.8 Errors with HTTPException

```python
from fastapi import HTTPException

FAKE_DB = {1: {"name": "policy.pdf"}}


@app.get("/documents/{doc_id}")
def get_document(doc_id: int):
    doc = FAKE_DB.get(doc_id)
    if doc is None:
        # Stops the function and sends: 404 {"detail": "Document not found"}
        raise HTTPException(status_code=404, detail="Document not found")
    return doc
```

**One handler for one type of error, for all endpoints:**

```python
from fastapi import Request
from fastapi.responses import JSONResponse
from openai import APIError


# If ANY endpoint raises an OpenAI APIError, FastAPI calls this function.
# The user gets a friendly 503 instead of an ugly 500 error.
@app.exception_handler(APIError)
async def llm_error_handler(request: Request, exc: APIError):
    return JSONResponse(
        status_code=503,
        content={"detail": "The AI service is busy. Please try again in a moment."},
    )
```

---

## 2.9 Lifespan: Create Things Once at Startup

Some things are slow to create: the LLM client, the Redis connection, a database pool, a reranker model.
Create them **once** when the server starts, and reuse them for every request.

```python
from contextlib import asynccontextmanager

import redis.asyncio as redis
from fastapi import FastAPI, Request
from openai import AsyncOpenAI


@asynccontextmanager
async def lifespan(app: FastAPI):
    # ---- Runs ONCE when the server starts ----
    app.state.llm = AsyncOpenAI()   # reads OPENAI_API_KEY from the environment
    app.state.redis = redis.from_url("redis://localhost:6379", decode_responses=True)

    yield   # <-- the server runs and handles requests while we are "paused" here

    # ---- Runs ONCE when the server stops ----
    await app.state.redis.aclose()
    await app.state.llm.close()


app = FastAPI(lifespan=lifespan)


@app.get("/ping-redis")
async def ping_redis(request: Request):
    # Use the shared client that was created at startup.
    return {"redis_ok": await request.app.state.redis.ping()}
```

> Old code uses `@app.on_event("startup")`. That still works but is **deprecated**. Use `lifespan`.

---

## 2.10 Background Tasks

Run small work **after** the response is sent, so the user does not wait for it.

```python
from fastapi import BackgroundTasks


def save_chat_log(user_id: str, question: str, answer: str):
    # Pretend we write to a database here. The user does not wait for this.
    print(f"Saved chat log for {user_id}")


@app.post("/ask")
async def ask(req: AskRequest, background_tasks: BackgroundTasks):
    answer = "Refunds are allowed within 30 days."

    # Schedule the function. It runs AFTER the response below is sent.
    background_tasks.add_task(save_chat_log, "user_1", req.question, answer)

    return {"answer": answer}
```

**BackgroundTasks vs a real queue:**

| | `BackgroundTasks` | Queue + workers (Celery, Service Bus, ...) |
|---|---|---|
| Where it runs | Same server process | Separate worker machines |
| If the server crashes | Task is **lost** | Job stays in the queue, gets retried |
| Good for | Small, quick jobs (logging, sending an email) | Heavy jobs (PDF ingestion, re-embedding) |

---

## 2.11 Streaming the LLM Answer

An LLM answer can take 5 seconds. If the user sees **nothing** for 5 seconds, it feels broken.
With **streaming**, we send each word as soon as the LLM makes it. The first word comes in under 1 second.
The total time is the same, but it **feels** much faster.

```python
from fastapi import Request
from fastapi.responses import StreamingResponse


@app.post("/ask/stream")
async def ask_stream(req: AskRequest, request: Request):
    llm = request.app.state.llm   # shared client from lifespan

    # A generator: each "yield" sends one piece of text to the user right away.
    async def token_generator():
        stream = await llm.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{"role": "user", "content": req.question}],
            stream=True,               # ask the LLM to send the answer piece by piece
        )
        async for chunk in stream:
            # Some chunks have no text (for example, the last one). Skip those.
            if chunk.choices and chunk.choices[0].delta.content:
                yield chunk.choices[0].delta.content

    return StreamingResponse(token_generator(), media_type="text/plain")
```

**SSE vs WebSocket:**

- **SSE (Server-Sent Events, `text/event-stream`)**: one direction, server → browser. Simple, normal HTTP. This is what ChatGPT-style chat apps usually use.
- **WebSocket**: two directions, both sides talk at any time. Use it for voice, live collaboration, games.

---

## 2.12 Middleware

**Middleware** is code that runs on **every** request, before and after your endpoint.
Good for: timing, request IDs, logging, CORS.

```python
import time
import uuid

from fastapi import Request


@app.middleware("http")
async def add_timing_and_request_id(request: Request, call_next):
    request_id = str(uuid.uuid4())   # unique ID to follow this request in the logs
    start = time.perf_counter()

    response = await call_next(request)   # run the actual endpoint

    duration_ms = (time.perf_counter() - start) * 1000
    response.headers["X-Request-ID"] = request_id
    print(f"{request.method} {request.url.path} -> {response.status_code} "
          f"in {duration_ms:.0f} ms (id={request_id})")
    return response
```

**CORS** (allow your website, on a different domain, to call the API from the browser):

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://mychatapp.com"],   # only our frontend, never "*" in production
    allow_methods=["*"],
    allow_headers=["*"],
)
```

---

## 2.13 File Upload

```python
from fastapi import File, HTTPException, UploadFile


@app.post("/documents")
async def upload_document(file: UploadFile = File(...)):
    # Check the file type before doing anything else.
    if file.content_type != "application/pdf":
        raise HTTPException(status_code=400, detail="Only PDF files are allowed")

    data = await file.read()   # read the file bytes (fine for small files)

    # Next steps (see Part 3): save to blob storage, then put a job in a queue.
    return {"filename": file.filename, "size_bytes": len(data)}
```

> File uploads need the `python-multipart` package. `pip install "fastapi[standard]"` includes it.

---

## 2.14 Testing

```python
# test_main.py   (run with: pytest)
from fastapi.testclient import TestClient

from main import app

client = TestClient(app)   # a fake browser that calls our app directly, no server needed


def test_health():
    r = client.get("/health")
    assert r.status_code == 200
    assert r.json() == {"status": "ok"}


def test_ask_rejects_short_question():
    # "hi" has 2 characters, but the model says min_length=3.
    r = client.post("/ask", json={"question": "hi"})
    assert r.status_code == 422
```

Tip: in tests, **never call the real LLM**. It is slow, costs money, and gives different answers each time.
Replace it with a fake using `app.dependency_overrides` or a mock.

---

## 2.15 Running in Production

**Development:**

```bash
uvicorn main:app --reload        # auto-restart when code changes
```

**Production (one machine):**

```bash
# 4 worker processes. Each one has its own event loop.
# Rule of thumb: number of workers ≈ number of CPU cores.
uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4
```

On Linux you may also see Gunicorn managing Uvicorn workers:

```bash
gunicorn main:app -k uvicorn.workers.UvicornWorker -w 4 -b 0.0.0.0:8000
```

**In containers (Kubernetes, Azure Container Apps):** use 1 or 2 workers per container, and scale by running **more containers**.

**Dockerfile:**

```dockerfile
# Small official Python image
FROM python:3.12-slim

WORKDIR /app

# Install packages first. Docker caches this step, so rebuilds are faster.
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy our code
COPY . .

# Start the server
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

# Part 3: Scenario: Scale a RAG System to 1M Users

## 3.1 What We Are Building

**"AskDocs"**: a chatbot for companies. Employees ask questions. The system finds the right parts of the company's PDFs and an LLM writes the answer, with sources.

It has **two flows**:

1. **Ingestion flow (write path)**: upload PDF → parse text → split into chunks → embed → save in vector DB.
2. **Query flow (read path)**: question → embed → search vector DB → (rerank) → build prompt → LLM → answer.

The **query flow** runs millions of times a day. The ingestion flow runs only when documents change.
So the query flow is the main thing to scale.

---

## 3.2 Step 1: Ask Questions First

**Never jump straight into drawing boxes.** First ask the interviewer:

- How many users? **Total** users, **daily** active users, or users **at the same time**?
- How many questions does a user ask per day?
- How fast must the answer be? (For example: first word in under 1.5 s.)
- How many documents? How often do they change?
- One company or many companies (**multi-tenant**)? Must their data be kept apart?
- Do we use a hosted LLM API (OpenAI / Azure OpenAI) or run our own model on GPUs?
- Do we need chat history? Streaming?
- What uptime do we promise? What is the budget?

**Our assumptions** (say them out loud and write them down):

| Item | Value |
|---|---|
| Registered users | 1,000,000 |
| Daily active users | 20% → 200,000 |
| Questions per active user per day | 10 |
| Peak traffic | 5 × the average |
| Speed goal (p95) | First word < 1.5 s, full answer < 6 s |
| Knowledge base | 10 million chunks |
| LLM | Hosted API (Azure OpenAI / OpenAI) |
| Tenants | Many companies, data must be separate |
| Uptime | 99.9% |

---

## 3.3 Step 2: Do the Math

**Traffic:**

```
Questions per day  = 200,000 users × 10 questions   = 2,000,000 per day
Average QPS        = 2,000,000 / 86,400 seconds      ≈ 23 per second
Peak QPS           = 23 × 5                          ≈ 115 per second
Design for         ≈ 200 QPS (extra room for safety)
```

**Requests open at the same time** (Little's Law: `in-flight = QPS × seconds per request`):

```
200 requests/second × 5 seconds each = 1,000 requests open at the same time
```

> **Lesson:** 200 QPS is not huge. The problem is that **each request is slow** (the LLM takes seconds).
> So we need servers that can keep **many requests waiting cheaply**. That is exactly what **async** FastAPI does.

**LLM tokens:**

```
Tokens per question = ~3,000 in (instructions + 5 chunks + question) + ~300 out ≈ 3,300
Tokens per day      = 2,000,000 × 3,300                         ≈ 6.6 billion
Tokens per minute   = 200 QPS × 3,300 × 60 seconds              ≈ 40 million at peak
```

> **Lesson:** the LLM is the **biggest cost** and the **biggest limit**.
> One LLM deployment has a tokens-per-minute (TPM) limit that is usually far below 40 million.
> So we need **several deployments**, **caching**, and **shorter prompts**.

**Vector storage:**

```
10,000,000 chunks × 1,536 numbers per vector × 4 bytes ≈ 61 GB (just the vectors)
```

> **Lesson:** too big for the memory of one small machine.
> Split it across machines (**shards**), or compress the vectors (**quantization**).

**Chat history:**

```
2,000,000 Q&A per day × ~2 KB ≈ 4 GB per day ≈ 1.5 TB per year
```

> **Lesson:** use a database that handles many writes, and move old chats to cheaper storage.


---

## 3.4 Step 3: Version 1, One Server

Always start simple. Then improve it.

```
   User ──► FastAPI (one server) ──► Vector DB (on the same machine)
               │
               └──────────────────► LLM API (OpenAI / Azure OpenAI)
```

This works for a demo or about 100 users. It is cheap and easy to build.

---

## 3.5 Step 4: What Breaks When Users Grow

| Problem | Why it happens | Fix |
|---|---|---|
| Server crashes and **everyone** is down | Only one server (single point of failure) | Many servers + load balancer |
| Slow when 1,000 users ask together | One machine, or blocking code | Async code + more servers (horizontal scaling) |
| A PDF upload makes chat slow | Ingestion runs on the same server | Queue + separate workers |
| LLM returns **429** errors | We hit the provider's TPM limit | More deployments, retries, user rate limits, caching |
| Huge LLM bill | Same questions are asked again and again | Caching, smaller model for easy questions |
| Vector search gets slow | 10M chunks on one machine | Managed vector DB with shards, replicas and an ANN index |
| User stares at a blank screen for 6 s | The LLM is slow | Streaming |
| Chat history lost on restart | Stored in server memory | Store in Redis / database (stateless servers) |
| Company A sees Company B's data | No filter on searches | Filter by `tenant_id` on **every** search and cache key |
| Nobody knows why it is slow | No monitoring | Logs, metrics, traces, alerts |

---

## 3.6 Step 5: Final Design for 1M Users

```
                 Users (web app / mobile app)
                               │
                               ▼
              ┌─────────────────────────────────┐
              │   API Gateway + Load Balancer   │  HTTPS, login check,
              │                                 │  rate limit, routing
              └────────────────┬────────────────┘
                               │
                               ▼
              ┌─────────────────────────────────┐
              │   FastAPI servers (stateless)   │  3 to 50 copies,
              │   [pod] [pod] [pod] ... [pod]   │  autoscale on load
              └────────────────┬────────────────┘
                               │
       ┌──────────────┬────────┴───────┬────────────────┐
       ▼              ▼                ▼                ▼
   ┌────────┐   ┌───────────┐    ┌──────────┐    ┌─────────────┐
   │ Redis  │   │ Vector DB │    │ Chat DB  │    │ LLM Gateway │
   └────────┘   └───────────┘    └──────────┘    └─────────────┘
   cache,       chunks +         users,          many LLM
   rate limits  vectors          chat history    deployments


   INGESTION PATH (separate, so uploads never slow down chat)

   Upload API ──► Blob Storage ──► Queue ──► Workers ──► Vector DB
                  (raw PDFs)                 (parse, chunk, embed;
                                              autoscale on queue size)

   EVERYWHERE: logs + metrics + traces ──► dashboards + alerts
```

**Which cloud product for which box:**

| Box | General tool | Azure | AWS |
|---|---|---|---|
| Gateway / load balancer | Nginx, Kong | Front Door, API Management | ALB, API Gateway |
| FastAPI servers | Kubernetes | AKS, Container Apps | EKS, ECS |
| Cache | Redis | Azure Managed Redis (or Azure Cache for Redis) | ElastiCache |
| Vector DB | Qdrant, pgvector, Pinecone | Azure AI Search | OpenSearch |
| Chat DB | PostgreSQL, MongoDB | Cosmos DB, Azure Database for PostgreSQL | DynamoDB, RDS |
| File storage | Object storage | Blob Storage | S3 |
| Queue | RabbitMQ, Kafka | Service Bus | SQS |
| LLM | OpenAI API | Azure OpenAI | Bedrock |
| Monitoring | Prometheus + Grafana | Azure Monitor, Application Insights | CloudWatch |

---

## 3.7 One Request, Step by Step

The user types: **"What is our refund policy?"**

1. The browser sends `POST /ask/stream` with the user's login token.
2. The **load balancer** sends it to a healthy FastAPI server.
3. The server checks the token: who is this user, and which company (`tenant_id`)?
4. **Rate limit** check in Redis: more than 20 questions in the last minute? → return 429.
5. **Answer cache** check in Redis: did someone in the same company ask this recently? → return the saved answer (about 5 ms, free).
6. **Embed** the question (check the embedding cache first).
7. **Search** the vector DB, **filtered by `tenant_id`** → top 20 chunks.
8. (Optional) **Rerank** → keep the best 5.
9. **Build the prompt**: instructions + 5 chunks + question.
10. **Call the LLM** with streaming. On 429 or timeout → retry or use another deployment. Still failing → fallback model.
11. **Stream** the words back to the user.
12. **After the answer**: save it in the cache, save chat history (in the background), log tokens and latency.

**Where does the time go?** (rough numbers)

| Step | Time |
|---|---|
| Login + rate limit + cache check | 5 to 10 ms |
| Embed the question | 50 to 150 ms |
| Vector search | 20 to 100 ms |
| Rerank | 100 to 300 ms |
| LLM first word (TTFT) | 300 to 1,000 ms |
| LLM full answer | 2 to 6 s |

> **Lesson:** the LLM is about **90% of the time**. Making everything else faster helps very little.
> The big wins are: **stream** the answer, **cache** answers, **shorter prompts**, and **smaller models** for easy questions.

---

## 3.8 How Each Part Scales

### A. API servers (FastAPI)

- **Stateless**: no user data in memory. So we can run 3 copies or 50 copies.
- **Autoscaling**: add copies when busy, remove them when quiet.
- **Scale on open requests, not CPU.** A RAG server is mostly **waiting** for the LLM, so its CPU stays low even when it is overloaded. Scaling on CPU would add servers too late.
- Keep **at least 3 copies** in different availability zones (different data centers), so one failure does not take the app down.

### B. Caching (the biggest cost saver)

| Layer | Key | Example hit rate |
|---|---|---|
| Answer cache (exact) | tenant + cleaned question text | Common questions ("How do I reset my password?") |
| Semantic cache | tenant + question **vector** | "refund rules?" ≈ "what is the refund policy?" |
| Embedding cache | hash of the text | Same text is never embedded twice |

If 30% of questions hit the cache, that is **30% fewer LLM calls** → about 30% lower LLM cost and much faster answers for those users.

**How a semantic cache works:**

1. Embed the new question.
2. Search a small "cache index" of past questions (same company only).
3. If a past question is **very** similar (for example, similarity > 0.95), return its saved answer.

**Be careful with semantic caching:**

- "Refund policy for **India**" and "refund policy for **USA**" look very similar but need different answers. Use a **high** threshold.
- **Always include `tenant_id`** in the cache key. Never give Company A an answer built from Company B's documents.
- Use a TTL, and **clear the cache when documents change**.

### C. Vector DB

- **ANN index (Approximate Nearest Neighbor, for example HNSW)**: instead of comparing the question with all 10M vectors, it checks only a few "neighborhoods". Very fast, and almost as accurate.
- **Replicas** → more searches per second (more QPS).
- **Shards (partitions)** → more data than one machine can hold.
- **Filter by `tenant_id`** on every search. This is for **security** and it also makes search faster.
- **Hybrid search** (keywords + vectors) gives better results for names, codes and IDs.
- Very big tenants can get their **own index**.

### D. LLM layer (the hardest part)

- **Many deployments / regions behind an LLM gateway** (for example LiteLLM or Azure API Management). The gateway spreads the load, so we are not stuck on one TPM limit.
- **Retries with backoff** on 429 / 5xx, and follow the `Retry-After` header.
- **Fallback model** if the main one fails.
- **Model routing**: small, cheap model for simple questions, big model only for hard ones.
- **Shorter prompts**: fewer, better chunks (rerank, then send 3 to 5), and a summary of old chat turns instead of the full history.
- **Limit the answer length** (`max_tokens`).
- **Reserved capacity** (for example, Azure OpenAI Provisioned Throughput) for steady, high traffic: you pay for guaranteed capacity.
- **Self-hosting** (for example vLLM on GPUs): scale by adding GPU servers. Request **batching** is how one GPU serves many users.

### E. Ingestion (workers)

- Upload API only **saves the file and adds a job to the queue**. It replies `202 Accepted` at once.
- **Autoscale workers on queue length**: long queue → more workers.
- **Batch embedding calls** (for example, 100 chunks per API call).
- **Idempotent**: processing the same document twice must not create duplicate chunks. Use IDs like `doc_id-chunk_number` and overwrite (upsert).
- Give ingestion its **own LLM / embedding quota**, so a big upload does not use up the quota chat users need.
- If a document has not changed (same file hash), skip it.

### F. Databases

- **Chat history**: partition by `user_id`. Write it in the background so the user does not wait.
- **Read replicas** for read-heavy data.
- Move chats older than, say, 90 days to cheap storage.

### G. Reliability

- Multiple availability zones, health checks, automatic restart of sick servers.
- Circuit breaker on the LLM and vector DB.
- **Graceful degradation**: if the LLM is down, show the top matching passages with a note: "AI answer is not available right now."
- Backups of the database and the vector index (you can rebuild the index from Blob Storage too).

### H. Security

- Login with JWT / OAuth (for example Microsoft Entra ID).
- **`tenant_id` filter on every search and every cache key.**
- Limit input size (`max_length`) to stop huge prompts.
- Guard against **prompt injection** (text in documents that tries to give the LLM new orders).
- Hide personal data (PII) in logs.
- Keep secrets in a vault (Azure Key Vault), never in code.

---

## 3.9 Code: A Scalable RAG API

This is a small but realistic version of the design above. Put **Parts A to G in one file**, `rag_api.py`, in this order.
The worker is a separate file.

Install:

```bash
pip install "fastapi[standard]" openai "redis>=5"
```

### Part A: Imports and settings

```python
# rag_api.py
import hashlib
import json
import os
import time
import uuid
from contextlib import asynccontextmanager

import redis.asyncio as redis
from fastapi import Depends, FastAPI, File, Header, HTTPException, Request, UploadFile
from fastapi.responses import StreamingResponse
from openai import APIError, AsyncOpenAI
from pydantic import BaseModel, Field

# Settings come from environment variables, never written in the code.
# The same code runs on your laptop and in production. Only the env values change.
REDIS_URL = os.getenv("REDIS_URL", "redis://localhost:6379")
CHAT_MODEL = os.getenv("CHAT_MODEL", "gpt-4o")              # main model
FALLBACK_MODEL = os.getenv("FALLBACK_MODEL", "gpt-4o-mini")  # backup if the main one fails
EMBED_MODEL = os.getenv("EMBED_MODEL", "text-embedding-3-small")

RATE_LIMIT_PER_MINUTE = 20         # questions per user per minute
CACHE_TTL_SECONDS = 60 * 60        # keep cached answers for 1 hour
TOP_K = 5                          # chunks to send to the LLM
MAX_ANSWER_TOKENS = 500            # limit answer length = limit cost and time
```

### Part B: Startup, shutdown and data models

```python
@asynccontextmanager
async def lifespan(app: FastAPI):
    # Create clients ONCE per server process and reuse them for every request.
    # Making a new client per request is slow and wastes connections.
    app.state.redis = redis.from_url(REDIS_URL, decode_responses=True)
    app.state.llm = AsyncOpenAI(
        timeout=30,      # never wait more than 30 s for the LLM
        max_retries=3,   # the SDK retries 429 / 5xx / timeouts with backoff for us
    )
    # For Azure OpenAI, use AsyncAzureOpenAI(azure_endpoint=..., api_version=...) instead.
    yield
    await app.state.redis.aclose()
    await app.state.llm.close()


app = FastAPI(title="AskDocs RAG API", lifespan=lifespan)


class AskRequest(BaseModel):
    question: str = Field(..., min_length=3, max_length=2000)


class AskResponse(BaseModel):
    answer: str
    sources: list[str]
    cached: bool = False


class User(BaseModel):
    user_id: str
    tenant_id: str   # which company this user belongs to


@app.get("/health")
async def health():
    # The load balancer calls this every few seconds.
    # If it fails, the load balancer stops sending traffic to this server.
    return {"status": "ok"}
```

### Part C: Login and rate limit (dependencies)

```python
async def get_current_user(authorization: str = Header(...)) -> User:
    # DEMO ONLY: the header looks like "Bearer user_1:acme".
    # In real life: verify a JWT token here and read user_id and tenant_id from it.
    token = authorization.removeprefix("Bearer ").strip()
    try:
        user_id, tenant_id = token.split(":")
    except ValueError:
        raise HTTPException(status_code=401, detail="Invalid token")
    return User(user_id=user_id, tenant_id=tenant_id)


async def rate_limit(request: Request, user: User = Depends(get_current_user)) -> User:
    # Fixed-window rate limit: count each user's requests per minute in Redis.
    # Redis is shared by ALL servers, so the limit is correct even with 50 servers.
    r = request.app.state.redis

    window = int(time.time() // 60)               # a new number every minute
    key = f"rate:{user.user_id}:{window}"

    count = await r.incr(key)                     # +1 for this request (starts at 1)
    await r.expire(key, 60)                       # old windows delete themselves

    if count > RATE_LIMIT_PER_MINUTE:
        raise HTTPException(status_code=429, detail="Too many questions. Please wait a minute.")
    return user
```

### Part D: Cache helpers

```python
def answer_cache_key(tenant_id: str, question: str) -> str:
    # Clean the text, so "What is X?" and "what  is x?" give the same key.
    clean = " ".join(question.lower().split())
    digest = hashlib.sha256(clean.encode()).hexdigest()
    # tenant_id is part of the key, so Company A never gets Company B's cached answer.
    return f"answer:{tenant_id}:{digest}"


async def get_cached(r, key: str) -> dict | None:
    data = await r.get(key)
    return json.loads(data) if data else None


async def set_cached(r, key: str, value: dict, ttl: int = CACHE_TTL_SECONDS):
    await r.set(key, json.dumps(value), ex=ttl)   # ex = expire after this many seconds
```

### Part E: The RAG steps (embed, search, build prompt, call LLM)

```python
async def embed(llm: AsyncOpenAI, r, text: str) -> list[float]:
    # Embedding cache: the same text always gives the same vector, so never pay twice.
    key = "emb:" + hashlib.sha256(text.encode()).hexdigest()
    cached = await r.get(key)
    if cached:
        return json.loads(cached)

    resp = await llm.embeddings.create(model=EMBED_MODEL, input=text)
    vector = resp.data[0].embedding
    await r.set(key, json.dumps(vector), ex=24 * 3600)   # keep for 1 day
    return vector


async def search_chunks(vector: list[float], tenant_id: str, top_k: int) -> list[dict]:
    # Replace this with your real vector DB call (Azure AI Search, Qdrant, pgvector, ...).
    # See RAG_Azure.md for a real Azure AI Search example.
    #
    # Two rules that matter at scale:
    #   1. ALWAYS filter by tenant_id, so users only see their own company's documents.
    #   2. Use an ANN index (like HNSW), so search stays fast with millions of chunks.
    return [
        {"text": "Refunds are allowed within 30 days of purchase.", "source": "policy.pdf p.4"},
    ]


def build_messages(question: str, chunks: list[dict]) -> list[dict]:
    # Put each chunk with its source name, so the LLM can cite it.
    context = "\n\n".join(f"[{c['source']}]\n{c['text']}" for c in chunks)
    return [
        {
            "role": "system",
            "content": (
                "Answer ONLY using the context. If the answer is not in the context, "
                "say you don't know. Cite sources in [brackets]."
            ),
        },
        {"role": "user", "content": f"Context:\n{context}\n\nQuestion: {question}"},
    ]


async def call_llm(llm: AsyncOpenAI, messages: list[dict]) -> str:
    # Try the main model first. If it still fails after the SDK's own retries,
    # try the smaller backup model, so the user gets an answer and not an error.
    for model in (CHAT_MODEL, FALLBACK_MODEL):
        try:
            resp = await llm.chat.completions.create(
                model=model,
                messages=messages,
                max_tokens=MAX_ANSWER_TOKENS,
                temperature=0.1,   # low = more factual, less creative
            )
            return resp.choices[0].message.content
        except APIError:
            continue   # this model failed, try the next one
    raise HTTPException(status_code=503, detail="The AI service is busy. Please try again.")
```

### Part F: The `/ask` endpoint (normal JSON answer)

```python
@app.post("/ask", response_model=AskResponse)
async def ask(req: AskRequest, request: Request, user: User = Depends(rate_limit)):
    # Depends(rate_limit) already checked login AND the rate limit before we get here.
    r = request.app.state.redis
    llm = request.app.state.llm

    # 1. Cache: did someone in this company ask the same thing recently? (~5 ms, free)
    key = answer_cache_key(user.tenant_id, req.question)
    cached = await get_cached(r, key)
    if cached:
        return AskResponse(**cached, cached=True)

    # 2. Retrieve: embed the question, then search ONLY this company's chunks.
    vector = await embed(llm, r, req.question)
    chunks = await search_chunks(vector, user.tenant_id, TOP_K)

    # 3. Generate the answer with the LLM.
    answer = await call_llm(llm, build_messages(req.question, chunks))

    # 4. Save it, so the next person who asks gets it instantly.
    result = {"answer": answer, "sources": [c["source"] for c in chunks]}
    await set_cached(r, key, result)
    return AskResponse(**result)
```

### Part G: The `/ask/stream` endpoint and the upload endpoint

```python
@app.post("/ask/stream")
async def ask_stream(req: AskRequest, request: Request, user: User = Depends(rate_limit)):
    r = request.app.state.redis
    llm = request.app.state.llm
    key = answer_cache_key(user.tenant_id, req.question)

    # Cache hit: send the whole saved answer at once.
    cached = await get_cached(r, key)
    if cached:
        return StreamingResponse(iter([cached["answer"]]), media_type="text/plain")

    vector = await embed(llm, r, req.question)
    chunks = await search_chunks(vector, user.tenant_id, TOP_K)
    messages = build_messages(req.question, chunks)

    async def generate():
        stream = await llm.chat.completions.create(
            model=CHAT_MODEL,
            messages=messages,
            max_tokens=MAX_ANSWER_TOKENS,
            stream=True,
        )
        parts = []
        async for chunk in stream:
            if chunk.choices and chunk.choices[0].delta.content:
                text = chunk.choices[0].delta.content
                parts.append(text)
                yield text            # send this piece to the user right away

        # The stream has finished: save the full answer in the cache.
        full = {"answer": "".join(parts), "sources": [c["source"] for c in chunks]}
        await set_cached(r, key, full)

    return StreamingResponse(generate(), media_type="text/plain")


async def save_to_blob(path: str, data: bytes):
    # Replace with Azure Blob Storage / S3 upload code.
    pass


@app.post("/documents", status_code=202)
async def upload_document(
    request: Request,
    file: UploadFile = File(...),
    user: User = Depends(get_current_user),
):
    # Do NOT process the PDF here. Parsing and embedding a big PDF can take minutes,
    # and it would slow down chat for everyone on this server.
    # Just store the file, add a job to the queue, and reply at once.
    if file.content_type != "application/pdf":
        raise HTTPException(status_code=400, detail="Only PDF files are allowed")

    doc_id = str(uuid.uuid4())
    path = f"{user.tenant_id}/{doc_id}.pdf"
    await save_to_blob(path, await file.read())

    job = {"doc_id": doc_id, "tenant_id": user.tenant_id, "path": path}
    await request.app.state.redis.lpush("ingest_jobs", json.dumps(job))   # add job to the queue

    # 202 Accepted = "we got it, we will process it soon"
    return {"doc_id": doc_id, "status": "queued"}
```

### The worker (separate program)

```python
# worker.py
# Runs as a SEPARATE program, NOT inside the API. Run many copies of it.
# Start it with:  python worker.py
import asyncio
import json

import redis.asyncio as redis


async def process_document(job: dict):
    # 1. Download the PDF from blob storage (job["path"]).
    # 2. Extract the text (use OCR if the PDF is scanned).
    # 3. Split the text into chunks.
    # 4. Embed chunks in BATCHES (for example 100 per API call): fewer calls, faster.
    # 5. Upsert into the vector DB with tenant_id, using id = f"{doc_id}-{chunk_no}".
    #    Same id twice = overwrite, not duplicate. So it is safe to retry a job.
    # 6. Clear this company's answer cache, so old answers do not hide the new info.
    #    (Easy trick: keep a per-tenant version number inside the cache key, and add 1 to it.)
    print(f"Processed document {job['doc_id']}")


async def main():
    r = redis.from_url("redis://localhost:6379", decode_responses=True)
    print("Worker started. Waiting for jobs...")

    while True:
        # BRPOP waits (blocks) until a job arrives, up to 5 seconds. No busy looping.
        item = await r.brpop("ingest_jobs", timeout=5)
        if item is None:
            continue                          # no job yet, wait again

        _queue_name, raw_job = item
        job = json.loads(raw_job)
        try:
            await process_document(job)
        except Exception as e:
            # Keep failed jobs in a "dead letter" list, so we can check and retry them later.
            print(f"Failed {job['doc_id']}: {e}")
            await r.lpush("ingest_failed", raw_job)


if __name__ == "__main__":
    asyncio.run(main())
```

> A plain Redis list is fine for learning. In production, use a real queue (Celery, Azure Service Bus, SQS).
> They keep a job safe if a worker crashes in the middle of it.

### Run and deploy

```bash
# Local: start Redis, the API, and a worker (in 3 terminals)
docker run -p 6379:6379 redis
uvicorn rag_api:app --reload
python worker.py

# Try it
curl -X POST http://127.0.0.1:8000/ask \
  -H "Authorization: Bearer user_1:acme" \
  -H "Content-Type: application/json" \
  -d '{"question": "What is our refund policy?"}'
```

**Autoscaling in Kubernetes** (add more API copies when busy):

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: rag-api
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: rag-api
  minReplicas: 3          # never less than 3, for high availability
  maxReplicas: 50         # upper limit, to protect the budget
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60   # add pods when average CPU is above 60%
```

> Remember: RAG servers mostly **wait**, so CPU can stay low under heavy load.
> Scaling on **number of open HTTP requests** works better. KEDA (in Kubernetes) and Azure Container Apps support this.
> For workers, scale on **queue length**.

---

## 3.10 Saving Money

The LLM bill is usually the biggest cost. Ways to cut it:

1. **Cache** answers (exact + semantic). 30% hit rate on 2M questions/day = **600,000 fewer LLM calls per day**.
2. **Route** easy questions to a small, cheap model. Use the big model only when needed.
3. **Send fewer chunks**: rerank and keep the best 3 to 5, not 20.
4. **Short chat history**: summarize old turns instead of sending all of them.
5. **Limit answer length** with `max_tokens`.
6. **Prompt caching** by the provider: keep the long system prompt the same and at the start of the prompt. Repeated prompt starts are cheaper.
7. **Batch** embedding calls during ingestion. **Do not re-embed** unchanged documents.
8. **Rate limits per plan** (free users get fewer questions than paid users).
9. **Track cost per company**, so you know who uses the most.

---

## 3.11 Bottlenecks and Fixes

| Part | Sign of trouble | Fix |
|---|---|---|
| API servers | High latency, many open connections | More pods, async code, no blocking calls |
| LLM | Many 429 errors, slow first token | More deployments / regions, gateway, retries, caching, model routing, reserved capacity |
| Vector DB | Search p95 keeps growing | Replicas (more QPS), shards (more data), tune the ANN index, filters |
| Embedding | Ingestion is slow | Batch calls, more workers |
| Redis | Memory full | TTLs, LRU eviction, Redis cluster |
| Chat DB | Slow writes | Partition by user, write in the background, read replicas |
| Queue | Queue keeps getting longer | Autoscale workers on queue length |

---

# Part 5: One-Page Cheat Sheet

**System design in 6 steps:** Clarify → Estimate → Simple design → Scale it → Deep dive on the bottleneck → Monitoring, security, trade-offs.

**Magic numbers:**

- 1 day ≈ 86,400 s ≈ 100,000 s (easy for mental math)
- 1M questions per day ≈ 12 QPS on average
- Open requests at once = QPS × seconds per request (Little's Law)
- 99.9% uptime ≈ 43 minutes down per month
- 1 vector (1,536 numbers × 4 bytes) ≈ 6 KB → 10M vectors ≈ 61 GB

**Building blocks:**

| Need | Tool |
|---|---|
| Spread traffic | Load balancer |
| More capacity | Horizontal scaling of stateless servers |
| Faster + cheaper | Cache (answer, semantic, embedding) |
| Slow work | Queue + workers |
| Protect the system | Rate limiting (429) |
| Survive failures | Timeouts, retries with backoff, fallback, circuit breaker |
| See problems | Logs, metrics, traces, alerts |
| Find similar text | Vector DB with an ANN index (HNSW), replicas, shards |

**RAG at 1M users, in one line each:**

- Stateless async FastAPI behind a load balancer, autoscaled on open requests.
- Redis for answer / semantic / embedding cache and rate limits.
- Vector DB with replicas (QPS) and shards (size). Always filter by `tenant_id`.
- LLM gateway over several deployments, with retries, fallback, model routing, `max_tokens`.
- Stream answers to cut the perceived wait.
- Ingestion through blob storage + queue + autoscaled workers. Idempotent upserts.
- Watch p95, TTFT, errors, tokens, cost, cache hit rate, answer quality.

**FastAPI in one line each:**

- `@app.get / @app.post` → routes
- Pydantic `BaseModel` → input validation (422 on bad input) and `response_model` for output
- `async def` + `await` for I/O. Never block inside `async def`
- `Depends()` → login, DB, rate limit
- `HTTPException(status_code=..., detail=...)` → errors
- `lifespan` → create clients once at startup
- `BackgroundTasks` → small jobs after the response
- `StreamingResponse` → stream LLM tokens
- `TestClient` → tests
- `uvicorn main:app --workers N` → production
