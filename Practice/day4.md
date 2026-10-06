# Day 4 — Caching

Today we'll learn one of the **most important system-design concepts: caching**.

Don't worry about Redis or Google Cloud initially.

First understand the **problem** that caching solves.

Our goal today:

```text
Understand the problem
        ↓
Understand cache
        ↓
Build simple architecture
        ↓
Understand Cache-Aside
        ↓
Understand TTL
        ↓
Understand cache invalidation
        ↓
Connect to Google Cloud
        ↓
Apply caching to GenAI systems
```

---

# 1. Start With Yesterday's Architecture

Yesterday we had:

```text
                         USERS
                           |
                           ↓
                         CLIENT
                           |
                           ↓
                    LOAD BALANCER
                           |
                  ┌────────┼────────┐
                  ↓        ↓        ↓
               SERVER    SERVER    SERVER
                  └────────┼────────┘
                           |
                           ↓
                       DATABASE
```

Suppose our application has **10 million users**.

A user frequently asks:

> "Give me my profile."

The request looks like:

```text
User
 ↓
Server
 ↓
Database
 ↓
User
```

That works.

But now imagine the same user asks repeatedly:

```text
Get my profile
Get my profile
Get my profile
Get my profile
Get my profile
```

Every request goes to the database.

```text
                    SERVER
                      |
       ┌──────────────┼──────────────┐
       ↓              ↓              ↓
       DB             DB             DB
       ↑              ↑              ↑
    Request        Request        Request
```

We're repeatedly asking the database for the **same information**.

That is inefficient.

---

# 2. The Basic Problem

Imagine a restaurant.

You ask the waiter:

> "Give me the menu."

The waiter goes to the kitchen every single time.

You ask again:

> "Give me the menu."

He goes to the kitchen again.

Again:

> "Give me the menu."

Again he goes to the kitchen.

That makes no sense.

Instead, the waiter could keep a copy nearby.

Then:

```text
First request
     ↓
Kitchen
     ↓
Get menu
     ↓
Keep copy nearby
```

Next request:

```text
Customer
   ↓
Nearby copy
```

That's basically **caching**.

---

# 3. What Is a Cache?

A cache is:

> **A temporary, fast storage layer used to store frequently accessed data so that we don't have to repeatedly retrieve it from a slower source.**

Simple architecture:

```text
                 Server
                /      \
               ↓        ↓
            Cache    Database
```

The cache is usually much faster than going to the primary database for every repeated read.

---

# 4. Real-Life Example

Think about your browser.

Suppose you visit a website.

The browser may cache things like:

```text
Images
CSS
JavaScript
Other resources
```

When you visit again, the browser may already have some of those resources.

So instead of downloading everything again:

```text
Website
   ↓
Internet
   ↓
Download
```

it can use:

```text
Website
   ↓
Browser Cache
```

Much faster.

The same principle applies to backend systems.

---

# 5. Our Application Without Cache

Let's start here:

```text
User
 ↓
Server
 ↓
Database
 ↓
Server
 ↓
User
```

Suppose:

```text
Database response = 100 ms
```

and we receive:

```text
100,000 requests
```

If many requests ask for the same information, the database is doing unnecessary work.

---

# 6. Add a Cache

Now:

```text
User
 ↓
Server
 ↓
Cache
 ↓
Database
```

But there's an important question:

> **What happens when the data is already in the cache?**

If it is there:

```text
User
 ↓
Server
 ↓
Cache
 ↓
Response
```

We don't need to access the database.

---

# 7. Cache Hit

Suppose Ajit's profile is already in the cache.

```text
User
 ↓
Server
 ↓
Cache
 ↓
Ajit profile
```

This is called a:

> **Cache Hit**

Meaning:

> The requested data was found in the cache.

---

# 8. Cache Miss

Now suppose we request:

```text
Rahul's profile
```

but Rahul's profile isn't in the cache.

The request becomes:

```text
User
 ↓
Server
 ↓
Cache
 ↓
❌ Not found
 ↓
Database
```

This is called a:

> **Cache Miss**

The server then gets the data from the database.

Usually, we can then put that data into the cache:

```text
Database
   ↓
User data
   ↓
Cache
```

Now the next request can be served from the cache.

---

# 9. The Most Important Cache Pattern

This is called:

> **Cache-Aside Pattern**

The flow is:

```text
             Request
                |
                ↓
              Server
                |
                ↓
              Cache
             /     \
         HIT       MISS
          |          |
          ↓          ↓
       Return      Database
                     |
                     ↓
                   Cache
                     |
                     ↓
                  Return
```

Let's understand it slowly.

---

# 10. Cache-Aside — Step by Step

Suppose user requests:

```text
GET /users/123
```

### Step 1

Server checks cache:

```text
Cache.get("user:123")
```

---

### Step 2 — Cache Hit

If the user exists:

```text
Cache
 ↓
User data
 ↓
Server
 ↓
User
```

Done.

---

### Step 3 — Cache Miss

If user doesn't exist:

```text
Cache
 ↓
❌
```

Server goes to database:

```text
Server
 ↓
Database
```

---

### Step 4

Database returns:

```text
User 123
Ajit
ajit@gmail.com
```

---

### Step 5

Server stores the result in cache:

```text
Database
 ↓
Cache
```

Now:

```text
Cache

user:123 → Ajit
```

---

### Step 6

Return response:

```text
Server
 ↓
User
```

---

# 11. Complete Cache-Aside Architecture

```text
                         USER
                           |
                           ↓
                         CLIENT
                           |
                           ↓
                      LOAD BALANCER
                           |
                           ↓
                        SERVER
                           |
                           ↓
                         CACHE
                       /       \
                    HIT         MISS
                     |            |
                     ↓            ↓
                  Return       DATABASE
                                  |
                                  ↓
                                Data
                                  |
                                  ↓
                                CACHE
                                  |
                                  ↓
                               Return
```

This architecture is extremely important.

You should be able to explain it without memorizing the diagram.

---

# 12. Why Is Cache Faster?

At a simple level:

```text
Cache
 ↓
Fast
```

while:

```text
Database
 ↓
Comparatively slower
```

The exact performance depends on the systems involved, network, workload, etc.

But conceptually:

```text
Cache → fast access
Database → persistent storage
```

That's why we don't replace the database with a cache.

---

# 13. Very Important:

## Cache ≠ Database

This is a common beginner mistake.

You might think:

> "If cache is faster, why don't we just use cache?"

Because cache is generally **temporary storage**.

For example:

```text
Database
    ↓
Permanent / durable data
```

while:

```text
Cache
    ↓
Temporary copy
```

If cache disappears:

```text
Cache ❌
```

we should still have:

```text
Database ✅
```

We can rebuild the cache.

---

# 14. Example

Database:

```text
user:123 → Ajit
```

Cache:

```text
user:123 → Ajit
```

Now cache crashes.

```text
Cache ❌
```

Database still has:

```text
user:123 → Ajit
```

Next request:

```text
Server
 ↓
Cache
 ↓
MISS
 ↓
Database
 ↓
Ajit
 ↓
Cache
```

Cache is rebuilt.

---

# 15. What Should We Cache?

We don't cache everything.

Good candidates are often:

```text
Frequently accessed
+
Relatively expensive to retrieve
+
Acceptable to be slightly stale
```

Examples:

```text
User profiles
Product information
Popular articles
Configuration
Session information
Frequently accessed API responses
```

---

# 16. What Should We NOT Cache?

Suppose you are processing:

```text
Bank transaction
```

You probably don't want to casually cache the authoritative transaction result and treat it as the source of truth.

For critical transactional data, the database remains the source of truth.

Similarly, highly dynamic data may not benefit much from caching.

The key question is:

> **Can this data safely be temporarily copied?**

---

# 17. The Big Problem: Stale Data

Now suppose the database says:

```text
Ajit
Age = 30
```

Cache also says:

```text
Ajit
Age = 30
```

Then Ajit updates his age:

```text
Database
Age = 31
```

But cache still contains:

```text
Age = 30
```

Now we have:

```text
Database → 31
Cache    → 30
```

This is called:

> **Stale data**

The cache contains an old version of the data.

---

# 18. Cache Invalidation

How do we solve this?

We need a strategy to remove/update stale cache entries.

This is called:

> **Cache invalidation**

Suppose Ajit updates his profile.

We can do:

```text
Update Database
       ↓
Delete/Update Cache
```

For example:

```text
User updates profile
       ↓
Database updated
       ↓
Cache entry deleted
```

Then the next read causes:

```text
Cache
 ↓
MISS
 ↓
Database
 ↓
New data
 ↓
Cache
```

---

# 19. Why Do People Say:

> "Cache invalidation is hard"?

Because keeping two copies synchronized can become complicated.

Imagine:

```text
Database
    ↓
Cache 1
Cache 2
Cache 3
Cache 4
```

Now data changes.

We need to make sure the old value doesn't remain somewhere unexpectedly.

This is why cache invalidation is an important system-design topic.

---

# 20. TTL

Another important concept:

> **TTL = Time To Live**

It means:

> How long should an item remain in the cache?

Example:

```text
TTL = 5 minutes
```

Suppose we store:

```text
user:123 → Ajit
```

at:

```text
10:00 AM
```

It might automatically expire at:

```text
10:05 AM
```

Then:

```text
Cache
 ↓
Expired
 ↓
MISS
 ↓
Database
```

---

# 21. Why Use TTL?

Because it automatically removes old data.

For example:

```text
Product price
```

Suppose product information can change.

We could say:

```text
Cache for 5 minutes
```

After 5 minutes:

```text
Cache expires
```

Next request gets fresh information.

---

# 22. TTL Doesn't Solve Everything

Important.

Suppose:

```text
TTL = 1 hour
```

but the database value changes after:

```text
5 minutes
```

The cache might still contain the old value for another 55 minutes.

So TTL is useful, but it is not always enough.

We may also explicitly invalidate/update the cache when data changes.

---

# 23. Cache Eviction

Imagine cache has limited memory.

Suppose it can hold:

```text
1 million entries
```

But we try to add:

```text
1,000,001
```

What happens?

We need to remove something.

This is called:

> **Eviction**

One common strategy is:

> **LRU — Least Recently Used**

Meaning:

> Remove data that hasn't been used recently.

Conceptually:

```text
A → used recently
B → used recently
C → not used for long time
```

Cache is full.

It may remove:

```text
C
```

We'll study eviction policies later.

---

# 24. Now Let's Connect This to Google Cloud

The generic architecture is:

```text
Application
    |
    ↓
Cache
    |
    ↓
Database
```

On Google Cloud, a common managed caching service is:

> **Memorystore**

It supports managed in-memory data stores such as Redis.

Conceptually:

```text
                    Users
                      |
                      ↓
                 Cloud Run
                      |
                      ↓
                 Memorystore
                   /      \
                HIT       MISS
                 |          |
                 ↓          ↓
              Return     Cloud SQL
                            |
                            ↓
                       Update Cache
```

The exact architecture depends on the application, networking, and service choices.

---

# 25. Why Memorystore?

Because we want a managed in-memory caching layer.

Instead of managing everything ourselves:

```text
Install Redis
Configure Redis
Maintain Redis
Scale Redis
Monitor Redis
Handle failures
```

we can use a managed cloud service.

For your Google Cloud interview, remember:

```text
Cache
  ↓
Memorystore
```

But don't simply answer:

> "Use Memorystore."

Explain **why**:

> "I would use a managed in-memory cache such as Memorystore to reduce repeated database reads, lower latency, and reduce database load for frequently accessed data."

That's much stronger.

---

# 26. Now Let's Add Cache to Our Previous Architecture

Day 3:

```text
Users
 ↓
Load Balancer
 ↓
Servers
 ↓
Database
```

Day 4:

```text
                         USERS
                           |
                           ↓
                         CLIENT
                           |
                           ↓
                    LOAD BALANCER
                           |
                  ┌────────┼────────┐
                  ↓        ↓        ↓
               SERVER    SERVER    SERVER
                  └────────┼────────┘
                           |
                           ↓
                         CACHE
                           |
                    ┌──────┴──────┐
                    ↓             ↓
                  HIT            MISS
                    |             |
                    ↓             ↓
                 Return       DATABASE
                                  |
                                  ↓
                                CACHE
```

This is our first architecture with **multiple layers**.

---

# 27. Why Is Cache Usually Between Application and Database?

Because the application knows:

> "I need user 123."

It first asks:

```text
Cache:
Do you have user 123?
```

If yes:

```text
Return
```

If no:

```text
Ask Database
```

So:

```text
Application
    ↓
Cache
    ↓
Database
```

makes intuitive sense.

---

# 28. Let's Think About a Real Google Cloud System

Imagine:

> **Design a product catalog for an e-commerce application.**

Users frequently request:

```text
GET /products/123
```

We might design:

```text
                     Users
                       |
                       ↓
               Cloud Load Balancer
                       |
                       ↓
                    Cloud Run
                       |
                       ↓
                 Memorystore
                  /        \
                HIT        MISS
                 |           |
                 ↓           ↓
              Product     Cloud SQL
                           |
                           ↓
                       Memorystore
```

Why?

Because product information is frequently read.

For example:

```text
iPhone 17
Price
Description
Image URL
Specifications
```

may be requested many times.

Instead of repeatedly hitting the database:

```text
1 million requests
       ↓
Cloud SQL
```

we can serve many repeated reads from cache.

---

# 29. Cache and GenAI

Now we're getting closer to your target role.

Caching is extremely useful in GenAI systems.

Suppose users ask:

```text
"What is the leave policy?"
```

Many employees ask exactly the same question.

Without caching:

```text
User
 ↓
API
 ↓
RAG
 ↓
Gemini
 ↓
Response
```

Every time we call the model.

That can increase:

```text
Latency
Cost
Model load
```

With caching:

```text
User
 ↓
API
 ↓
Cache
 ├── HIT → Return previous answer
 │
 └── MISS
       ↓
      RAG
       ↓
     Gemini
       ↓
     Cache
       ↓
    Response
```

Now repeated queries can potentially be served much faster and more cheaply.

---

# 30. But AI Caching Is More Complicated

Suppose:

User 1:

> "What is the leave policy?"

User 2:

> "Can you tell me about the company's leave policy?"

Same meaning.

But different text.

A simple exact-match cache won't necessarily recognize them as the same request.

We might eventually consider:

```text
Exact query cache
        ↓
Semantic cache
```

A semantic cache can use embeddings/similarity to determine whether a previous answer is sufficiently relevant.

**Don't implement this yet.**

Just understand the concept.

This will become important later when we design production GenAI systems.

---

# 31. Another GenAI Use Case: Embedding Cache

Suppose we have:

```text
Document
 ↓
Chunk
 ↓
Embedding Model
 ↓
Vector
```

Generating embeddings can also have a cost.

If the exact same content is processed repeatedly, we may be able to reuse an existing embedding.

Conceptually:

```text
Text
 ↓
Embedding Cache
 ├── HIT → Existing embedding
 │
 └── MISS → Embedding Model
```

Again, we're only learning the design concept today.

---

# 32. Cache Is Not Always Good

This is important for interviews.

If interviewer asks:

> "Should we add a cache?"

Don't automatically say yes.

Ask:

### Is the data read frequently?

If no:

```text
Cache may not help much.
```

### Can stale data be tolerated?

If no:

```text
Caching becomes more complicated.
```

### Is the database actually the bottleneck?

If no:

```text
Maybe cache isn't necessary.
```

### Is the extra infrastructure worth the complexity?

Because cache introduces:

```text
Cost
Complexity
Invalidation problems
Consistency challenges
```

So:

> **Caching is a trade-off, not a default requirement.**

---

# 33. Important Terms From Today

You should know these before moving on.

### Cache

Fast temporary storage.

### Cache Hit

Data found in cache.

```text
Cache → FOUND
```

### Cache Miss

Data not found.

```text
Cache → NOT FOUND → Database
```

### Cache-Aside

Application checks cache first, then database if needed.

### TTL

How long cached data remains valid.

### Cache Invalidation

Removing/updating stale cached data.

### Eviction

Removing cache entries because of limited capacity.

### LRU

Least Recently Used.

---

# 34. The Architecture You Should Remember

Don't memorize the exact diagram.

Understand this flow:

```text
                    REQUEST
                       |
                       ↓
                    SERVER
                       |
                       ↓
                     CACHE
                    /     \
                  HIT     MISS
                   |        |
                   ↓        ↓
                RESPONSE  DATABASE
                              |
                              ↓
                            CACHE
                              |
                              ↓
                           RESPONSE
```

This is **Cache-Aside**.

---

# 35. Interview Question

Imagine the interviewer says:

> **"Your database is getting millions of read requests. How would you reduce the load?"**

A weak answer:

> "I'll use Redis."

A better answer:

> "If many of those requests are repeated reads for data that doesn't change frequently, I would introduce a caching layer in front of the database. The application can use a cache-aside pattern: check the cache first, return on a cache hit, and on a miss read from the database and populate the cache. I would also define an appropriate TTL and an invalidation strategy based on how fresh the data needs to be."

That's system-design thinking.

---

# 36. Google Cloud Interview Version

If they specifically ask:

> "How would you implement this on Google Cloud?"

You could say:

> "For a managed caching layer, I would consider Memorystore. The application service, for example running on Cloud Run, would check Memorystore before querying the primary database such as Cloud SQL. This can reduce database load and improve latency for frequently accessed data."

Notice the order:

```text
Problem
 ↓
Solution
 ↓
Pattern
 ↓
Google Cloud service
```

**Not:**

```text
Google service
 ↓
Google service
 ↓
Google service
```

This distinction will matter a lot in your interviews.

---

# 37. Day 4 Practice

Before we move to Day 5, answer these in your own words.

### Q1

What is a cache?

### Q2

Why do we need a cache if we already have a database?

### Q3

What is a cache hit?

### Q4

What is a cache miss?

### Q5

Explain the Cache-Aside pattern.

### Q6

What happens when cached data becomes old?

### Q7

What is TTL?

### Q8

What is cache invalidation?

### Q9

Why can't we use cache as our only database?

### Q10

What Google Cloud service can we consider for managed caching?

### Q11 — GenAI

Suppose 10,000 users ask the exact same question:

> "What is our company leave policy?"

How could caching reduce:

```text
LLM cost
LLM calls
Latency
```

?

### Q12 — Architecture

Explain this architecture in your own words:

```text
                      USER
                        |
                        ↓
                     CLIENT
                        |
                        ↓
                   CLOUD RUN
                        |
                        ↓
                  MEMORystore
                   /        \
                HIT          MISS
                 |             |
                 ↓             ↓
              RETURN       CLOUD SQL
                                |
                                ↓
                           MEMORystore
```

---

## Your progress so far

You now have:

```text
DAY 1
What is System Design?
        ↓
DAY 2
Client → API → Server → Database
        ↓
DAY 3
SQL / NoSQL / Object Storage
        ↓
DAY 4
Cache / Cache-Aside / TTL
```

Next we'll add another major building block:

# Day 5 — Load Balancing + Scaling Deep Dive

We'll go much deeper into:

```text
1 server
   ↓
10 servers
   ↓
Load Balancer
   ↓
Health Checks
   ↓
Horizontal Scaling
   ↓
Auto Scaling
   ↓
Stateless Servers
   ↓
Google Cloud Load Balancing
   ↓
Cloud Run Auto Scaling
```

And most importantly, you'll learn **why stateless architecture is so important for scalable GenAI/Agentic AI systems**.
