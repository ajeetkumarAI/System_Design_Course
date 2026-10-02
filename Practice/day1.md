### The learning path

**Level 0 — What is System Design?**

* What is a system?
* What is an architecture?
* Client → API → backend → database
* Request/response
* Components and their responsibilities
* Functional vs non-functional requirements

**Level 1 — Build tiny systems**

1. Simple REST API
2. API + database
3. API + cache
4. File upload system
5. Authentication system
6. Notification system

You’ll learn concepts like:

* Load balancer
* API gateway
* Database
* Cache
* Queue
* Object storage
* Horizontal scaling
* Monitoring

**Level 2 — Cloud system design**
We'll map the concepts to Google Cloud:

| General concept | Google Cloud                     |
| --------------- | -------------------------------- |
| Compute         | Cloud Run / GKE / Compute Engine |
| API Gateway     | API Gateway                      |
| Object storage  | Cloud Storage                    |
| Relational DB   | Cloud SQL                        |
| NoSQL           | Firestore / Bigtable             |
| Data warehouse  | BigQuery                         |
| Cache           | Memorystore                      |
| Message queue   | Pub/Sub                          |
| Kubernetes      | GKE                              |
| Secrets         | Secret Manager                   |
| Monitoring      | Cloud Monitoring                 |
| AI platform     | Vertex AI                        |
| Vector search   | Vertex AI Vector Search          |
| GenAI           | Gemini on Vertex AI              |

**Level 3 — AI system design**
We'll gradually build:

`User → API → LLM`

then:

`User → API → LLM → Database`

then:

`Documents → Chunking → Embedding → Vector DB → Retrieval → LLM`

Then production RAG:

`User`
↓
`API Gateway`
↓
`Cloud Run`
↓
`Query Processing`
↓
`Vector Search`
↓
`Retriever`
↓
`Gemini`
↓
`Response`

Then we'll add:

* Authentication
* Authorization
* caching
* conversation memory
* metadata filtering
* hybrid search
* reranking
* citations
* evaluation
* guardrails
* observability
* cost control
* latency optimization
* high availability

**Level 4 — Agentic AI**

We'll build:

`User → Agent → Tools → APIs / DB / Search`

Then:

`Supervisor Agent`
↓
`Planner`
↓
`Specialist Agents`
↓
`Tools`
↓
`Enterprise systems`

We'll cover:

* Tool calling
* Function calling
* Agent state
* Planning
* Memory
* Human-in-the-loop
* Multi-agent architecture
* MCP
* Agent security
* Agent observability
* Agent evaluation
* Failure handling

**Level 5 — Interview-level architecture**

Eventually you'll be able to answer questions such as:

* Design a production RAG system for 10M documents.
* Design an enterprise GenAI chatbot.
* Design a customer-support AI agent.
* Design a multi-agent IT operations platform.
* Design a document intelligence system.
* Design a real-time AI fraud detection system.
* Design a Gemini-powered enterprise assistant.
* Design an AI coding assistant.
* Design an agent that can execute enterprise workflows.
* Design a multimodal RAG system.
* Design a globally scalable GenAI platform.

And importantly, you'll learn **how to think**, rather than memorizing diagrams.

---

# Lesson 1 — What exactly is System Design?

Forget AI for a moment.

Imagine I ask:

> **Design a food-delivery application.**

You don't start by saying:

> "I'll use Kubernetes, Kafka, Redis, MongoDB..."

That's the wrong starting point for a beginner.

First ask:

> **What does the system need to do?**

For example:

A user should be able to:

```text
1. Open the application
2. See restaurants
3. Select food
4. Place an order
5. Pay
6. Track the order
```

Now we have a **system**.

---

# What is a system?

A system is simply:

> **Multiple components working together to accomplish a goal.**

For example:

```text
User
  ↓
Application
  ↓
Backend
  ↓
Database
```

That's already a system.

Don't worry about scalability yet.

---

# What is architecture?

Architecture is simply:

> **How the different components of our system are connected and how they work together.**

For example:

```text
             ┌─────────────┐
             │    User     │
             └──────┬──────┘
                    │
                    ↓
             ┌─────────────┐
             │   Backend   │
             └──────┬──────┘
                    │
                    ↓
             ┌─────────────┐
             │  Database   │
             └─────────────┘
```

This is an architecture.

Very simple.

---

# Your first architecture

Let's design something extremely small.

### Problem

> Design a system where a user can enter their name and get a greeting.

For example:

```text
User:
Ajit
```

Response:

```text
Hello Ajit
```

We could build:

```text
             User
               │
               │ HTTP Request
               ↓
        ┌───────────────┐
        │   Web/API     │
        │    Server     │
        └───────┬───────┘
                │
                ↓
        "Hello " + name
                │
                ↓
             User
```

That's a system design.

No database.

No Kubernetes.

No AI.

No cloud.

---

# Now let's add a database

Suppose the requirement changes:

> We need to remember every user.

Now our system needs storage.

```text
             User
               │
               ↓
        ┌───────────────┐
        │   API Server  │
        └───────┬───────┘
                │
                │ save user
                ↓
        ┌───────────────┐
        │   Database    │
        └───────────────┘
```

Now imagine:

```text
User
  │
  │ POST /users
  ↓
API Server
  │
  │ INSERT
  ↓
Database
```

The database's job is:

> Store information permanently.

The API server's job is:

> Receive requests and perform business logic.

Already you've learned two fundamental system-design components.

---

# Now the important question

Suppose 10 users use our application.

One server is enough.

```text
       10 users
          │
          ↓
    ┌───────────┐
    │   Server  │
    └─────┬─────┘
          ↓
       Database
```

But what if:

```text
10 users
```

becomes:

```text
10,000 users
```

or:

```text
1,000,000 users
```

Now we have a **scaling problem**.

This is where system design becomes interesting.

---

# Scaling

Suppose one server can handle:

```text
1,000 requests/second
```

But our application receives:

```text
10,000 requests/second
```

One server isn't enough.

So we can use multiple servers.

```text
                    Users
                      │
                      ↓
               ┌─────────────┐
               │Load Balancer│
               └──────┬──────┘
                      │
          ┌───────────┼───────────┐
          ↓           ↓           ↓
      Server 1     Server 2    Server 3
          │           │           │
          └───────────┼───────────┘
                      ↓
                  Database
```

Now the load is distributed.

This is called **horizontal scaling**.

---

# One very important interview concept

You will hear this constantly:

### Vertical scaling

Make one machine bigger.

```text
Small Server
     ↓
Large Server
```

For example:

```text
4 CPU
8 GB RAM
```

becomes:

```text
32 CPU
128 GB RAM
```

---

### Horizontal scaling

Add more machines.

```text
Server 1
Server 2
Server 3
Server 4
```

instead of:

```text
One very powerful server
```

In modern cloud architecture, horizontal scaling is extremely important.

---

# Why do we need a Load Balancer?

Imagine you have:

```text
Server 1
Server 2
Server 3
```

Who decides which server receives the request?

The **load balancer**.

```text
                Users
                  │
                  ↓
          ┌───────────────┐
          │ Load Balancer │
          └───────┬───────┘
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
    Server 1   Server 2   Server 3
```

For example:

```text
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1
Request 5 → Server 2
```

The exact routing strategy can vary, but the basic idea is:

> **Distribute incoming traffic across multiple servers.**

---

# Now let's connect this to Google Cloud

Once you understand the general architecture, Google Cloud becomes much easier.

Our architecture:

```text
Users
  ↓
Load Balancer
  ↓
Multiple Servers
  ↓
Database
```

Could become something like:

```text
Users
  ↓
Google Cloud Load Balancing
  ↓
Cloud Run
  ↓
Cloud SQL
```

Conceptually:

```text
                    Users
                      │
                      ↓
             ┌─────────────────┐
             │ Cloud Load      │
             │ Balancing       │
             └────────┬────────┘
                      │
                      ↓
             ┌─────────────────┐
             │    Cloud Run    │
             │   API Service   │
             └────────┬────────┘
                      │
                      ↓
             ┌─────────────────┐
             │    Cloud SQL    │
             │    Database     │
             └─────────────────┘
```

Notice what happened.

We didn't memorize a Google Cloud diagram.

We first understood:

```text
User
 ↓
Load Balancer
 ↓
Servers
 ↓
Database
```

Then we mapped it to:

```text
User
 ↓
Cloud Load Balancing
 ↓
Cloud Run
 ↓
Cloud SQL
```

**This is exactly how I want you to learn system design.**

---

# The 6 questions you should eventually ask in every interview

When an interviewer gives you:

> "Design an AI-powered customer support system."

Don't immediately draw boxes.

Start thinking:

### 1. What are we building?

Understand the problem.

### 2. Who uses it?

For example:

```text
Customers
Support agents
Administrators
```

### 3. What does it need to do?

These are **functional requirements**.

Example:

```text
User asks question
       ↓
System understands question
       ↓
Search knowledge
       ↓
Generate answer
       ↓
Return answer
```

### 4. How big is the system?

This is where we ask about scale:

```text
100 users?
10,000?
10 million?
```

### 5. What can go wrong?

For example:

```text
Database unavailable
LLM unavailable
Network failure
Bad user input
LLM hallucination
Too many requests
```

### 6. What are our quality requirements?

These are **non-functional requirements**.

For example:

```text
Low latency
High availability
Security
Scalability
Reliability
Cost
Observability
```

These six questions will become second nature.

---

# Your eventual GenAI system-design framework

For your Google Cloud AI Engineer preparation, we'll eventually train you to approach a question like this:

```text
              1. Requirements
                     ↓
              2. Scale
                     ↓
              3. APIs
                     ↓
              4. High-level architecture
                     ↓
              5. Data storage
                     ↓
              6. AI/ML architecture
                     ↓
              7. Scaling
                     ↓
              8. Reliability
                     ↓
              9. Security
                     ↓
             10. Observability
                     ↓
             11. Cost
                     ↓
             12. Trade-offs
```

For an AI system, we'll additionally ask:

```text
Which model?
Which embedding model?
Which vector database?
How is retrieval done?
How do we handle hallucination?
How do we evaluate answers?
How do we handle prompt injection?
How do we secure tools?
How do we monitor LLM latency/cost?
How do we handle model failures?
```

---

## Our progression

I recommend we go in this exact order:

```text
PART 1
System Design Fundamentals
        ↓
PART 2
Client / API / Server / Database
        ↓
PART 3
Load Balancer + Scaling
        ↓
PART 4
Caching
        ↓
PART 5
Queues + Async Processing
        ↓
PART 6
Storage + Databases
        ↓
PART 7
Cloud Architecture
        ↓
PART 8
Basic AI System
        ↓
PART 9
RAG System
        ↓
PART 10
Production RAG
        ↓
PART 11
LLM Application
        ↓
PART 12
AI Agents
        ↓
PART 13
Multi-Agent Systems
        ↓
PART 14
MCP + Tool Security
        ↓
PART 15
Enterprise AI Architecture
        ↓
PART 16
Google Cloud AI System Design
        ↓
PART 17
Mock Google System Design Interviews
```


**Problem → Why do we need this? → Simple real-world example → Architecture → Every box → Request flow → Failure scenarios → Scaling → Google Cloud equivalent → Interview question.**

### Next lesson

We'll start with **"Client → API → Server → Database"** and build our **first real system from scratch**, using a tiny **User Registration System**.
