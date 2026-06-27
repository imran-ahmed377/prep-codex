# Table of Content

- [1. React](#1-react)
- [2. TypeScript](#2-typescript)
- [3. Node.js](#3-nodejs)
- [4. Express](#4-express)
- [5. Next.js](#5-nextjs)
- [6. Redis](#6-redis)
- [7. CrewAI](#7-crewai)
- [8. Prompt Chaining](#8-prompt-chaining)
- [9. AutoGen](#9-autogen)


# 1. React

## What is it?

React is a JavaScript library for building **user interfaces (UI)**—the parts of a website that users see and interact with.

Think of React as **LEGO blocks** for websites.

Instead of building an entire webpage from scratch, you create reusable pieces (called **components**).

### For example

```text
Website

├── Navbar
├── Search Box
├── Product Card
├── Shopping Cart
└── Footer
```

Each of these is a React component.

## Example

Imagine an Amazon product page.

Instead of writing the product card 100 times:

```text
Product Card

📷 Image
Title
Price
Buy Button
```

You create **one React component**.

Then React displays it for every product.

```jsx
<ProductCard />
<ProductCard />
<ProductCard />
<ProductCard />
```

## Real-life use cases

- Facebook
- Instagram
- Netflix
- Airbnb
- Spotify

Nearly every modern web app uses React or a similar UI library.

---

# 2. TypeScript

## What is it?

TypeScript is **JavaScript with safety checks**.

Imagine writing code like writing an email.

JavaScript lets you send the email without checking for mistakes.

TypeScript is like spell-check.

## Example

### JavaScript

```javascript
let age = 25;

age = "hello";
```

This runs (although it may later cause bugs).

TypeScript says:

> "Wait... age should be a number."

```typescript
let age: number = 25;

age = "hello"; // ❌
```

It catches the mistake before your program runs.

## Why companies use it

Imagine 100 developers working on one application.

TypeScript prevents many common mistakes before the software is deployed.

---

# 3. Node.js

## What is it?

Node.js lets JavaScript run **outside the browser**.

Normally JavaScript runs here:

```text
Browser

Chrome

Firefox

Safari
```

Node lets JavaScript run on the server:

```text
Your Computer

↓

Backend Server

↓

Database
```

## Example

User visits a website.

```text
Browser

↓

Node.js Server

↓

Database

↓

Returns data
```

Without Node, JavaScript couldn't easily power the server side of many web applications.

## Real-life use case

When you log into Netflix:

```text
Browser

↓

Node Server

↓

Checks username/password

↓

Returns your account
```

---

# 4. Express

## What is it?

Express is a framework that makes building servers with Node.js much easier.

Think of it like this:

```text
Node.js = Engine

Express = Car
```

You could build a car from just an engine—but Express gives you many useful features out of the box.

## Example

Suppose someone visits:

```text
www.shop.com/products
```

Express handles that request.

```javascript
app.get("/products", (req, res) => {
    res.send(products);
});
```

Meaning:

> If someone asks for `/products`, send them the product list.

## Use cases

- Login APIs
- Shopping cart APIs
- Payment services
- Mobile app backends

---

# 5. Next.js

## What is it?

Next.js is built on top of React.

React builds the user interface.

Next.js adds features like:

- Routing
- Backend APIs
- Faster page loading
- Better SEO
- Authentication support

Imagine building a house.

React gives you:

- Bricks
- Doors
- Windows

Next.js also gives you:

- Plumbing
- Electricity
- Roof
- Garage

## Example

Instead of creating routes manually,

```text
pages/

home.js
about.js
contact.js
```

Next.js automatically creates:

```text
/

about

/contact
```

No extra setup.

## Companies using Next.js

- TikTok
- Hulu
- Twitch
- OpenAI (parts of its web experience)

---

# 6. Redis

## What is it?

Redis is an **extremely fast in-memory database**.

Think of two storage options.

### Hard Drive

```text
Cabinet

↓

Slow

↓

Keeps data permanently
```

### Redis

```text
Sticky Note

↓

Very Fast

↓

Usually temporary
```

Redis stores data in RAM, making access much faster than reading from disk.

## Example

Suppose 10,000 users request the same weather information.

### Without Redis

```text
Every request

↓

Database

↓

Slow
```

### With Redis

```text
First request

↓

Database

↓

Save in Redis

↓

Future requests

↓

Redis

↓

Very Fast
```

## Common uses

- Login sessions
- Caching
- Rate limiting
- Chat applications
- Leaderboards

---

# 7. CrewAI

## What is it?

CrewAI is a framework for creating **multiple AI agents that work together**.

Imagine a company.

```text
CEO

↓

Manager

↓

Developer

↓

Tester

↓

Writer
```

Each employee has a different job.

CrewAI lets you build AI systems with specialized roles.

## Example

```text
Research Agent

↓

Coding Agent

↓

Testing Agent

↓

Documentation Agent
```

Each agent contributes to the overall task.

## Example use case

**Goal:**

> Build a travel itinerary.

```text
Research Agent

↓

Find flights

↓

Planner Agent

↓

Create itinerary

↓

Budget Agent

↓

Calculate cost

↓

Writer Agent

↓

Generate final report
```

---

# 8. Prompt Chaining

## What is it?

Instead of asking the AI to do everything at once, you break a complex task into smaller prompts.

## Example

### Without chaining

```text
Write an article.
```

### With chaining

```text
Prompt 1

Generate outline

↓

Prompt 2

Write introduction

↓

Prompt 3

Write body

↓

Prompt 4

Review grammar

↓

Prompt 5

Summarize
```

Each step improves the next.

## Example

Building a business plan.

```text
Generate ideas

↓

Evaluate ideas

↓

Create roadmap

↓

Estimate costs

↓

Write proposal
```

Each prompt builds on the previous one.

---

# 9. AutoGen

## What is it?

AutoGen is another framework for building **AI agents that collaborate**.

You define different agents and let them talk to each other.

## Example

```text
User

↓

Planner Agent

↓

Programmer Agent

↓

Tester Agent

↓

Reviewer Agent
```

The agents exchange messages until the work is complete.

## Example

You say:

> Build a calculator.

```text
Planner

↓

We need addition, subtraction, multiplication.

↓

Programmer

↓

Here is the code.

↓

Tester

↓

Multiplication fails.

↓

Programmer

↓

Fixed.

↓

Reviewer

↓

Looks good.
```

## Difference between CrewAI and AutoGen

They have similar goals—coordinating multiple AI agents—but different designs.

### CrewAI

- Emphasizes defined roles, tasks, and workflows.
- Easy to model teams with clear responsibilities.

### AutoGen

- Emphasizes conversations between agents.
- Agents negotiate, critique, and refine work through message exchanges.