# End to End Embedding Steps

## Step 1: Document Extraction

- **What you do:** You use a code library (like PyPDF, pdfplumber, or LangChain) to open your 1000 PDF pages.
- **The goal:** Strip out the raw text from the pages, separating it from the design and layout.

## Step 2: Chunking (Slicing the Cake)

- **What you do:** You cannot turn an entire 1000-page document into a single vector—it is too much data for the AI to handle. You must cut the text into small, bite-sized pieces called **chunks**.
- **The Golden Rule:** A standard chunk size is about **500 to 1000 words per chunk** (roughly 1 or 2 paragraphs). You also overlap them slightly (e.g., 50 words) so a sentence doesn't get cut in half at the border of a chunk.
- *Result:* Your 1000 PDF pages will turn into roughly **3,000 separate text chunks**.

### How to Decide Chunk Size (Paragraph Length)

A chunk is measured in **tokens** (which roughly equal 3/4 of a word).

- **The Default Choice (512 tokens / ~400 words):** This is a good starting point for many apps. It is long enough to hold a complete thought or paragraph, but short enough to keep retrieval focused.
- **When to Go Smaller (128–256 tokens):** Use smaller chunks if your PDFs are full of short, distinct facts, such as a dictionary, product catalog, FAQ, or a list of employee rules.
- **When to Go Larger (1000+ tokens):** Use larger chunks if your PDFs contain long, academic arguments or legal contract clauses where separating individual sentences would cause important context to be lost.

### How to Decide Overlap Size (The Safety Net)

Overlap means copying a small portion from the end of Chunk 1 and placing it at the beginning of Chunk 2. This helps prevent sentences or ideas from being split across chunk boundaries.

- **The General Rule:** Start with an overlap of around **10% to 20% of your chunk size**.
- If your chunk size is **500 words**, your overlap could be around **50 to 100 words**.
- If you have highly technical content, such as complex formulas or code blocks, you may need more overlap so that important context is not separated.

#### Simple Starting Point

For a typical business PDF, you could start with:

```text
Chunk Size: 512 tokens (~400 words)
Overlap:    10–20% (~40–80 words)
```

## Step 3: Embedding (Translating into Numbers)

- **What you do:** You send all 3,000 text chunks to an **Embedding Model** (like OpenAI's `text-embedding-3-small`).
- **The goal:** The model reads each paragraph and converts it into a long string of numbers (a vector) that mathematically represents its exact meaning.

### How to Choose: `text-embedding-3-small` vs. `text-embedding-3-large`

OpenAI offers two primary embedding options. Think of them as a **Standard Map** vs. a **High-Definition Satellite Map**.

| Feature | `text-embedding-3-small` | `text-embedding-3-large` |
| --- | ---: | ---: |
| **Vector Dimensions** | 1,536 numbers | 3,072 numbers |
| **Accuracy** | Good / Great for standard text | Best / Better at capturing subtle meaning |
| **Cost** | **Lower cost** | **Higher cost** |
| **Database Size** | Uses less storage | Uses roughly 2× the vector storage at full dimensions |

#### Choose `text-embedding-3-small` if:

- You are building a general business chatbot, such as an HR assistant or customer-support FAQ system.
- Your documents are mostly standard business text.
- You want to keep embedding and vector-storage costs low.
- You plan to use a reranker, which can improve the quality of the final retrieved results.

#### Choose `text-embedding-3-large` if:

- Your documents contain complex technical terminology, such as legal contracts, medical research, or advanced engineering manuals.
- You need higher retrieval quality and are willing to pay more for it.
- You need strong multilingual retrieval across languages such as English, Japanese, French, and others.

## Step 4: Upserting (Saving to the Vector Database)

- **What you do:** You upload (or "upsert") these 3,000 vectors into your vector database (like Pinecone).
- **What gets stored:** For each chunk, Pinecone saves three things:
  1. An **ID** (e.g., `chunk_142`).
  2. The **Vector** (the string of numbers for searching).
  3. The **Metadata** (the actual raw text of that paragraph and the page number, so you can read it later).

---

# After Embadding: Searching for Answers

1. **The Question:** A user asks: *"What is the warranty policy on page 450?"*

2. **The Fast Search:** The system converts the question into a vector (a list of numbers). It sends that vector to Pinecone or Azure AI Search, which searches the 3,000 stored chunks and retrieves the **top 50 closest matches**.

3. **The Rerank:** The reranker examines those 50 chunks against the original question. It assigns each chunk a relevance score. For example:

   - Chunk from page 450 → **0.98**
   - Chunk from page 449 → **0.91**
   - Chunk from page 120 → **0.42**
   - Unrelated chunk → **0.08**

   The reranker then sorts the chunks from **most relevant to least relevant**.

4. **The Cutoff:** The system applies its selection rule to the reranked results. For example, if the top score is `0.98`, the system might keep only chunks that are within a certain relative relevance range of that top score.

   This could reduce the 50 retrieved chunks down to **1–5 highly relevant chunks**, depending on the query and the scores.

5. **The Generation:** The system takes those final relevant chunks and places their text into the prompt sent to the **LLM** (such as ChatGPT or Claude).

   For example:

   > *"Answer the user's question using only the following retrieved information: [relevant chunks]."*

6. **The Result:** The LLM reads the retrieved information and generates the final answer.

   For example:

   *"According to page 450, the warranty lasts for 2 years."*

---

# End to End RAG Steps

## Step 1: The User Asks a Question

- **What happens:** A user types a question into your app (e.g., *"What is our company's refund policy for broken items?"*).
- **Explanation:** The system immediately sends this question to an **Embedding Model** (like OpenAI or Cohere) to turn the words into a string of numbers called a vector.

## Step 2: The Fast Search (Retrieval)

- **What happens:** The system takes those numbers and searches your **Vector Database** (where all your company documents are stored as numbers).
- **Explanation:** It does a super-fast math scan across millions of pages. It looks for pages that have a similar mathematical pattern to the user's question.

## Step 3: Grabbing the Rough Draft (Top-K)

- **What happens:** The database quickly grabs a fixed "rough draft" pile of pages—usually the **top 25 to 50 matches**.
- **Explanation:** This step is built for speed, not perfection. The system collects a wide safety net of pages to guarantee the correct answer is hidden somewhere inside the pile.

## Step 4: The Deep Clean (Reranking)

- **What happens:** The system sends those 25 to 50 rough pages, along with the original question, to the **Reranking API** (like Cohere or Jina AI).
- **Explanation:** The reranker reads the text of each page very carefully against the question. It gives every single page a precise relevance score between `0.0` and `1.0`.

## Step 5: Applying the 15% Cutoff Rule (Dynamic-K)

- **What happens:** The system looks at the highest-scoring page (the #1 match) and calculates a cutoff line that is 10% to 15% lower than that score.
- **Explanation:** Instead of keeping all 50 pages, it drops any page that falls below this cutoff line. If the question is simple, it might keep only 1 or 2 pages. If the question is complex, it might keep 5 or 6 pages.

## Step 6: Feeding the Final AI (The Prompt)

- **What happens:** The system takes the final, trimmed down, perfectly ordered pages and pastes them into a prompt for the **Large Language Model (LLM)** (like ChatGPT or Claude).
- **Explanation:** The prompt looks something like: *"Using only these 3 specific pages of text, please answer this question for the user."*

## Step 7: The Final Answer is Delivered

- **What happens:** The LLM reads the perfect, junk-free information and writes a clear response.
- **Explanation:** Because the reranker filtered out all the confusing, unrelated data, the LLM generates a highly accurate answer quickly and without hallucinating (making things up).

---

# The End-to-End Workflow with Azure AI Search

If you build your app inside Microsoft Azure, the workflow becomes **much simpler** because Azure can handle almost every step automatically using its built-in features.

Here is what the **Azure AI Search Workflow** looks like:

### Phase 1: The One-Time Setup (Ingestion)

Instead of writing complex code to chop up your PDFs and turn them into numbers, Azure does it for you:

1. **Upload:** You drop your 1000 PDF pages into an **Azure Blob Storage** folder.
2. **Crack & Chunk:** You turn on Azure's **Document Cracking** feature. It automatically reads the PDFs and chops them into chunks using your chosen chunk/overlap sizes.
3. **Embed:** Azure has a native connection to Azure OpenAI. It automatically passes the chunks to `text-embedding-3-small` and saves the numbers directly into your **Azure AI Search Index**.

### Phase 2: The Live User Query (The RAG Loop)

When a user asks a question, Azure runs its combined search in a single API call:

```text
[User Question]
       ↓
1. Hybrid Search
   (Finds text keywords + vector numbers at the same time)
       ↓
2. Retrieve Top-K
   (Grabs the top 50 matches)
       ↓
3. Semantic Ranker
   (Azure's built-in AI reranks the 50 chunks)
       ↓
4. Top 3–5 Chunks
       ↓
Azure OpenAI GPT-4o
       ↓
Answer Generated
```