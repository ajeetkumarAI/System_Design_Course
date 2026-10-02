> **What is system design, why do we need it, and what are the basic concepts?**

Let's answer it as if you were asked this in an interview.

---

# 1. What is System Design?

A simple interview answer:

> **System design is the process of deciding how different components of a software system will work together to satisfy functional requirements such as features, and non-functional requirements such as scalability, reliability, security, performance, and cost.**

For example, suppose we are building an application.

At the simplest level:

```text
User
  ↓
Application
  ↓
Database
```

System design means deciding:

* What components do we need?
* What does each component do?
* How do they communicate?
* Where do we store data?
* How will the system handle more users?
* What happens when something fails?
* How do we make it secure?
* How do we monitor it?

---

# 2. What is a System?

A **system** is a group of components working together to solve a problem.

For example, a food delivery system might contain:

```text
User App
    ↓
API
    ↓
Order Service
    ↓
Database
    ↓
Payment Service
    ↓
Restaurant Service
```

All these components work together.

So:

> **System = multiple components working together to achieve a goal.**

---

# 3. What is Architecture?

Architecture describes **how those components are connected**.

For example:

```text
              User
                |
                ↓
          ┌──────────┐
          │   API    │
          └────┬─────┘
               |
               ↓
          ┌──────────┐
          │ Database │
          └──────────┘
```

This is a very simple architecture.

The user sends a request.

The API processes it.

The database stores or retrieves information.

---

# 4. Why can't we just use one server?

For a small application, we can.

```text
        Users
          |
          ↓
      One Server
          |
          ↓
       Database
```

Suppose the server can handle:

```text
1,000 requests/second
```

But eventually our application receives:

```text
10,000 requests/second
```

Now one server is not enough.

So we add more servers.

```text
                 Users
                   |
                   ↓
             Load Balancer
                   |
        ┌──────────┼──────────┐
        ↓          ↓          ↓
     Server 1   Server 2   Server 3
        |          |          |
        └──────────┼──────────┘
                   ↓
                Database
```

Now the workload can be distributed.

This introduces our first major system-design concept:

> **Scalability**

---

# 5. What is Scalability?

Scalability means:

> **The ability of a system to handle increasing workload by adding resources or changing the architecture.**

For example:

```text
100 users
   ↓
1,000 users
   ↓
10,000 users
   ↓
1 million users
```

We want our system to continue working as demand increases.

There are two basic approaches.

---

# 6. Vertical Scaling

Make the existing server more powerful.

For example:

```text
Before:

4 CPU
8 GB RAM
```

becomes:

```text
After:

32 CPU
128 GB RAM
```

Diagram:

```text
Small Server
     ↓
Bigger Server
```

This is:

> **Vertical scaling**

---

# 7. Horizontal Scaling

Instead of making one server bigger, add more servers.

```text
Server 1
Server 2
Server 3
Server 4
```

Diagram:

```text
             Users
               |
               ↓
        Load Balancer
               |
       ┌───────┼───────┐
       ↓       ↓       ↓
    Server 1 Server 2 Server 3
```

This is:

> **Horizontal scaling**

This concept is **very important for your Google Cloud interview** because cloud-native systems commonly use horizontal scaling.

---

# 8. What is a Load Balancer?

Imagine 100,000 people arrive at a restaurant.

There are three counters:

```text
Counter 1
Counter 2
Counter 3
```

Someone needs to distribute customers among those counters.

That's roughly what a load balancer does.

```text
                 Users
                   |
                   ↓
            Load Balancer
                   |
        ┌──────────┼──────────┐
        ↓          ↓          ↓
     Server 1   Server 2   Server 3
```

It receives incoming requests and distributes them across available servers.

So remember:

> **Load Balancer = distributes incoming traffic across multiple servers.**

---

# 9. What is a Database?

A database is used to **store and retrieve information**.

For example:

```text
User
----------------
ID
Name
Email
Password
```

Our architecture becomes:

```text
User
 ↓
API Server
 ↓
Database
```

Suppose the user sends:

```text
POST /users

Name = Ajit
Email = ajit@gmail.com
```

The API receives the request.

Then the API stores the information:

```text
Database

ID     Name      Email
1      Ajit      ajit@gmail.com
```

---

# 10. Functional Requirements

This is one of the most important interview concepts.

**Functional requirements describe what the system should do.**

For example, if the interviewer says:

> Design an AI customer-support system.

Functional requirements could be:

```text
1. User can ask a question.
2. System searches company documents.
3. System generates an answer.
4. System returns the answer.
5. User can see conversation history.
```

These are **features**.

Think:

> **WHAT should the system do?**

---

# 11. Non-Functional Requirements

Non-functional requirements describe **how well the system should work**.

For example:

```text
Scalability
Performance
Availability
Reliability
Security
Cost
Observability
```

Suppose our AI chatbot works like this:

```text
User
 ↓
AI System
 ↓
Answer
```

Functionally, it works.

But suppose it takes:

```text
60 seconds
```

to answer every question.

That's a problem.

So we care about:

> **Latency**

Maybe we want:

```text
Response < 3 seconds
```

That's a non-functional requirement.

---

# 12. Functional vs Non-Functional

This distinction is extremely important.

| Type           | Question                   |
| -------------- | -------------------------- |
| Functional     | What should the system do? |
| Non-functional | How well should it do it?  |

Example:

### Functional

```text
User can upload PDF.
```

### Non-functional

```text
PDF upload should complete within 5 seconds.
```

Another example:

### Functional

```text
User can ask an AI question.
```

### Non-functional

```text
System should support 100,000 concurrent users.
```

---

# 13. Now imagine a GenAI system

This is where your target role becomes relevant.

Suppose interviewer asks:

> **Design an enterprise AI chatbot.**

Don't immediately say:

```text
Vertex AI
Gemini
LangChain
Vector Search
Cloud Run
Pub/Sub
GKE
...
```

Instead, first understand the problem.

At the simplest level:

```text
              User
                |
                ↓
             API
                |
                ↓
              LLM
                |
                ↓
             Answer
```

That's our first AI system.

---

# 14. Now suppose the AI needs company documents

The LLM doesn't automatically know the company's private documents.

So we add document retrieval.

```text
                 User
                   |
                   ↓
                  API
                   |
                   ↓
              Retrieval
                   |
                   ↓
             Company Docs
                   |
                   ↓
                  LLM
                   |
                   ↓
                Answer
```

Now we've started moving toward **RAG**.

Eventually we'll build this properly:

```text
                 User
                   |
                   ↓
             API Gateway
                   |
                   ↓
               AI Service
                   |
          ┌────────┴─────────┐
          ↓                  ↓
     Vector Search        Gemini
          ↑                  ↑
          │                  │
          └──── Retrieved ───┘
                Context
```

But **don't worry about this architecture yet**.

We'll build it step by step.

---

# 15. The most important mental model

When you see any system-design question, think:

```text
                 REQUIREMENTS
                      ↓
                   USERS
                      ↓
                 API / ENTRY
                      ↓
                  SERVICES
                      ↓
                  STORAGE
                      ↓
                   SCALE
                      ↓
                 RELIABILITY
                      ↓
                   SECURITY
                      ↓
                OBSERVABILITY
                      ↓
                    COST
```

For AI systems, add:

```text
                 AI MODEL
                    ↓
                RETRIEVAL
                    ↓
                 PROMPTS
                    ↓
                TOOLS/APIs
                    ↓
                EVALUATION
                    ↓
              AI SECURITY
```

---

# 16. What you should NOT do in an interview

A beginner often does this:

Interviewer:

> Design an enterprise GenAI chatbot.

Candidate:

> I'll use Gemini, Vertex AI, LangChain, Cloud Run, Pub/Sub, Redis, PostgreSQL, Kubernetes, Kafka, Vector DB...

This is basically **technology dumping**.

The interviewer wants to understand:

> **Why did you choose each component?**

For example:

**Why Cloud Run?**

You should eventually be able to say:

> I need a scalable stateless service for the API/application layer, and Cloud Run provides managed container execution with automatic scaling.

**Why a vector database?**

> Because semantic search requires storing embeddings and retrieving documents based on vector similarity.

**Why Pub/Sub?**

> For asynchronous processing where the producer should not wait for the consumer to finish.

That's system-design thinking.

---

# 17. Your first interview answer

If the interviewer asks:

> **"What is system design?"**

You can answer:

> "System design is the process of designing the architecture of a software system by identifying the required components, their interactions, data flow, storage, and the decisions needed to meet functional and non-functional requirements such as scalability, performance, availability, reliability, security, and cost."

That's a good **interview-level definition**.

But I don't want you to memorize it blindly.

You should understand this:

```text
Problem
   ↓
Requirements
   ↓
Components
   ↓
Communication
   ↓
Data
   ↓
Scale
   ↓
Failures
   ↓
Security
   ↓
Monitoring
```

---

# 18. One final example

Let's say:

> **Design a simple website where users can register.**

You start with:

```text
             User
               |
               ↓
           API Server
               |
               ↓
            Database
```

Then interviewer says:

> We now have 1 million users.

You think:

```text
             Users
               |
               ↓
         Load Balancer
               |
       ┌───────┼───────┐
       ↓       ↓       ↓
    Server   Server   Server
       └───────┼───────┘
               ↓
           Database
```

Then interviewer says:

> Database is receiving too many requests.

Now you start thinking about:

```text
Caching
Database scaling
Read replicas
Database partitioning
```

Then:

> Some operations don't need to happen immediately.

You think:

```text
Queue
   ↓
Background Worker
```

Then:

> We need the system to work even if one server fails.

You think:

```text
Multiple servers
Health checks
Failover
```

Then:

> We need to deploy this on Google Cloud.

You map those concepts to Google Cloud services.

**That progression is system design.**

---

## What I want you to remember from this lesson

Don't memorize 50 technologies.

For now, remember these **10 words**:

```text
1. User
2. API
3. Server
4. Database
5. Load Balancer
6. Scaling
7. Cache
8. Queue
9. Security
10. Monitoring
```

And this basic picture:

```text
                         USERS
                           |
                           ↓
                    LOAD BALANCER
                           |
                  ┌────────┼────────┐
                  ↓        ↓        ↓
               SERVER   SERVER   SERVER
                  └────────┼────────┘
                           ↓
                        DATABASE
```

Once this becomes comfortable, we'll start adding **one component at a time**.

**Next lesson:** we'll build the first system — **User Registration System** — from an empty page. We'll go through the request **line by line**, including what happens when you click "Register", before introducing any complicated cloud terminology.
