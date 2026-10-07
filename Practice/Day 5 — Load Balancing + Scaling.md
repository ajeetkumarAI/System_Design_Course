# Day 5 — Load Balancing + Scaling

Today we will understand one of the **most important system design topics**:

> What happens when 1 server is not enough?

We will learn:

1. Load Balancer
2. Health Checks
3. Horizontal Scaling
4. Vertical Scaling
5. Auto Scaling
6. Stateless Servers
7. Google Cloud mapping
8. Why this is important for GenAI systems

---

# 1. Start with ONE server

Suppose we build a simple application.

```text
Users
  |
  v
Server
  |
  v
Database
```

Assume our server can handle:

```text
1 server → 1,000 requests/second
```

Now suddenly:

```text
5,000 requests/second
```

What happens?

```text
Users
  |
  v
Server
  |
  X
Overloaded
```

The server may become:

* slow
* unresponsive
* CPU 100%
* memory exhausted
* requests start timing out
* eventually the application may crash

So we need more servers.

---

# 2. Add multiple servers

We can add 5 servers:

```text
                 +----------+
                 | Server 1  |
                 +----------+
                      |
Users  ---> ??? ------+
                      |
                 +----------+
                 | Server 2  |
                 +----------+
                      |
                 +----------+
                 | Server 3  |
                 +----------+
                      |
                 +----------+
                 | Server 4  |
                 +----------+
                      |
                 +----------+
                 | Server 5  |
                 +----------+
```

But now we have another problem.

## Who decides which server receives the request?

That's the job of a **Load Balancer**.

---

# 3. What is a Load Balancer?

A load balancer is like a **traffic police officer**.

Imagine 5 roads going to the same destination.

Instead of sending every car to Road 1:

```text
Cars
 |
 v
Road 1
Road 1
Road 1
Road 1
Road 1
```

Traffic becomes terrible.

A traffic controller distributes cars:

```text
              +--> Road 1
              |
Cars ---> Traffic Controller
              |
              +--> Road 2
              |
              +--> Road 3
              |
              +--> Road 4
              |
              +--> Road 5
```

A Load Balancer does the same thing with requests.

```text
                         +----------+
                    ---> | Server 1 |
                   /     +----------+
                  /
Users ---> Load Balancer ---> Server 2
                  \
                   \     +----------+
                    ---> | Server 3 |
                          +----------+
```

---

# 4. Why do we need a Load Balancer?

Suppose 3 servers exist.

```text
Server 1 → 1,000 req/sec
Server 2 → 1,000 req/sec
Server 3 → 1,000 req/sec
```

Total capacity:

```text
3 × 1,000
=
3,000 requests/sec
```

The Load Balancer distributes traffic.

For example:

```text
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1
Request 5 → Server 2
Request 6 → Server 3
```

This is one simple form of distribution.

---

# 5. What if one server crashes?

Very important interview question.

Suppose:

```text
                  +----------+
              --->| Server 1 |  OK
             /    +----------+
            /
Users ---> LB ---> Server 2   ❌ CRASHED
            \
             \    +----------+
              --->| Server 3 |  OK
                  +----------+
```

Should the Load Balancer continue sending requests to Server 2?

Obviously:

**No.**

But how does it know?

Through a **Health Check**.

---

# 6. Health Check

The Load Balancer periodically checks whether servers are healthy.

For example:

```text
Load Balancer
     |
     | "Are you healthy?"
     |
     v
Server 1 → "YES"
Server 2 → "NO"
Server 3 → "YES"
```

The Load Balancer removes Server 2 from traffic.

```text
Users
  |
  v
Load Balancer
  |
  +------> Server 1
  |
  +------> Server 3

Server 2 ❌
```

This improves **availability**.

---

# 7. Horizontal Scaling

This is extremely important.

Suppose we have:

```text
1 server
```

and it is overloaded.

We add more servers:

```text
1 server
   ↓
3 servers
   ↓
10 servers
   ↓
50 servers
```

This is called:

# Horizontal Scaling

Think:

> Horizontal = add more machines.

Example:

```text
Before:

+--------+
| Server |
+--------+


After:

+--------+  +--------+  +--------+
|Server 1|  |Server 2|  |Server 3|
+--------+  +--------+  +--------+
```

---

# 8. Vertical Scaling

Another approach is to make the existing server bigger.

For example:

```text
Before:

4 CPU
16 GB RAM
```

Upgrade it:

```text
After:

16 CPU
64 GB RAM
```

This is:

# Vertical Scaling

Think:

> Vertical = make one machine bigger.

Diagram:

```text
Small Server
     |
     | Upgrade
     v
Big Server
```

---

# 9. Horizontal vs Vertical

Very important interview comparison.

| Horizontal                         | Vertical                            |
| ---------------------------------- | ----------------------------------- |
| Add more servers                   | Make server bigger                  |
| 1 → 10 servers                     | 4 CPU → 32 CPU                      |
| Better for distributed systems     | Simpler architecture                |
| Can provide better fault tolerance | One machine can remain a bottleneck |
| More operational complexity        | Hardware limits exist               |

For modern cloud applications, **horizontal scaling is very common**.

---

# 10. Auto Scaling

Now imagine traffic changes throughout the day.

Morning:

```text
100 requests/sec
```

Afternoon:

```text
1,000 requests/sec
```

Evening:

```text
10,000 requests/sec
```

Should we permanently run 20 servers?

Maybe not.

It would waste money during low traffic.

Instead:

```text
Low traffic
    ↓
3 servers

High traffic
    ↓
20 servers
```

The system automatically adds/removes capacity.

This is:

# Auto Scaling

---

# 11. Simple Auto Scaling Example

Suppose:

```text
CPU > 70%
```

We add another server.

```text
3 servers
    |
    | CPU becomes high
    v
4 servers
```

If traffic decreases:

```text
4 servers
    |
    | CPU becomes low
    v
3 servers
```

So:

```text
Traffic increases
       ↓
Need more capacity
       ↓
Add servers
```

And:

```text
Traffic decreases
       ↓
Less capacity required
       ↓
Remove servers
```

This gives us:

* scalability
* better resource utilization
* potentially lower cost

---

# 12. Now an important concept: Stateless Servers

This is one of the most important concepts for scalable system design.

Suppose we have:

```text
User
 |
 v
Server 1
```

Server 1 remembers:

```text
User = Ajeet
Cart = Laptop
```

Now the next request goes to Server 2:

```text
User
 |
 v
Server 2
```

But Server 2 doesn't know anything about the user.

Problem.

---

# 13. Don't store important user state inside one server

Instead of:

```text
Server 1
  |
  +-- User session
  +-- Shopping cart
  +-- Conversation state
```

we can store shared state externally.

For example:

```text
                 +----------+
                 | Server 1 |
                 +----------+
                      |
                      |
                 +----------+
                 | Database |
                 +----------+
                      |
                 +----------+
                 | Server 2 |
                 +----------+
```

Now any server can access the state.

```text
User
 |
 v
Load Balancer
 |
 +----> Server 1 ----+
 |                   |
 +----> Server 2 ----+----> Shared Storage
 |                   |
 +----> Server 3 ----+
```

This allows requests to go to **any healthy server**.

That is the basic idea behind a **stateless application server**.

---

# 14. Why Stateless Architecture is Important

Imagine:

```text
10 servers
```

If every server has its own private user state:

```text
Server 1 → User A data
Server 2 → User B data
Server 3 → User C data
...
```

Scaling becomes difficult.

But if state is stored in shared systems:

```text
Servers
  |
  v
Shared Database / Cache / Storage
```

then:

```text
Server 1
Server 2
Server 3
Server 4
Server 5
```

can all handle requests.

This makes horizontal scaling much easier.

---

# 15. Very important: Stateless does NOT mean "no state"

This confuses many beginners.

Stateless means:

> The application server should not depend on its own local memory for important persistent user state.

State can still exist.

It can be stored in:

* Database
* Cache
* Object storage
* Session store

For example:

```text
Application Server
       |
       +----> Cloud SQL
       |
       +----> Memorystore
       |
       +----> Cloud Storage
```

---

# 16. Google Cloud Mapping

Now let's connect today's concepts to GCP.

A basic architecture could look like:

```text
Users
  |
  v
Google Cloud Load Balancing
  |
  v
Cloud Run
  |
  +------> Cloud SQL
  |
  +------> Memorystore
  |
  +------> Cloud Storage
```

Cloud Run can run multiple instances of your application and scale based on incoming traffic.

Conceptually:

```text
                 +----------------+
                 |  Load Balancer |
                 +----------------+
                         |
                         v
              +---------------------+
              |      Cloud Run      |
              +---------------------+
                /        |        \
               /         |         \
              v          v          v
          Instance 1  Instance 2  Instance 3
```

The important system-design idea is not to memorize every product detail.

Understand:

```text
Traffic
   ↓
Load Balancing
   ↓
Multiple application instances
   ↓
Shared data systems
```

---

# 17. Now bring GenAI into the picture

This is where today's lesson becomes important for your Google AI Engineer interview.

Suppose we have a GenAI application:

```text
User
  |
  v
API
  |
  v
Gemini
  |
  v
Response
```

One server may work for a demo.

But imagine:

```text
10,000 users
```

Now:

```text
User
User
User
User
...
10,000 users
       |
       v
   API Layer
       |
       v
Gemini
```

We need scalable architecture.

A simplified production architecture:

```text
                     Users
                       |
                       v
                Load Balancer
                       |
                       v
                API / Cloud Run
                 /     |      \
                /      |       \
               v       v        v
          Instance  Instance  Instance
               \       |       /
                \      |      /
                 v     v     v
                 Cache
                   |
                   v
              RAG / Services
                   |
          +--------+--------+
          |                 |
          v                 v
      Vector Store       Gemini
```

Now we are starting to think like a system designer.

---

# 18. Interview Question

Suppose interviewer asks:

> "Your GenAI application works perfectly with 100 users. Suddenly 10,000 users start using it. What would you do?"

Don't immediately say:

> "Use Kubernetes."

Instead, think systematically.

### Step 1 — Find the bottleneck

Ask:

```text
Is API server overloaded?
Is database overloaded?
Is vector search slow?
Is Gemini latency high?
Is network slow?
Is cache missing?
Is concurrency too high?
```

### Step 2 — Scale application layer

Use:

```text
Load Balancer
      ↓
Multiple application instances
```

### Step 3 — Auto scale

```text
Traffic ↑
   ↓
Instances ↑
```

### Step 4 — Reduce unnecessary work

Use caching where appropriate.

```text
Request
  ↓
Cache
  ↓
HIT → return
MISS → continue
```

### Step 5 — Check AI bottlenecks

For GenAI:

```text
LLM latency
Token usage
Concurrency
Rate limits
Cost
```

This is much stronger than simply saying:

> "I'll add more servers."

---

# 19. Your System Design Mental Model

At this point, your architecture thinking should start becoming:

```text
                  USER
                    |
                    v
              Load Balancer
                    |
                    v
             Application/API
             /      |       \
            /       |        \
           v        v         v
        Cache    Database   Other Services
                              |
                              v
                           AI Model
```

And then ask:

### Traffic

```text
How many users?
How many requests/sec?
```

### Scaling

```text
Can I add more instances?
```

### Availability

```text
What if one instance dies?
```

### Performance

```text
Where is the bottleneck?
```

### Data

```text
Where is persistent state stored?
```

### Cache

```text
Can I avoid repeated expensive work?
```

### AI

```text
What happens when the LLM gets 10,000 requests?
```

---

# Day 5 — Practice Questions

Try answering these yourself before looking at the next lesson.

### Basic

1. What problem does a Load Balancer solve?

2. What happens if one server crashes?

3. What is a health check?

4. What is horizontal scaling?

5. What is vertical scaling?

6. What is auto scaling?

7. Why is horizontal scaling useful?

8. What does stateless server mean?

9. Does stateless mean there is no user state?

10. Where can application state be stored?

### Interview level

11. You have one server handling 1,000 requests/sec. Suddenly traffic becomes 10,000 requests/sec. Design a solution.

12. You have 10 servers behind a Load Balancer. One server crashes. What happens?

13. Why should application servers preferably be stateless?

14. Your GenAI application suddenly gets 10,000 users. What components would you investigate first?

15. How would you scale a GenAI API?

---

## Today's architecture to remember

Don't memorize the diagram. Understand the flow:

```text
                         USERS
                           |
                           v
                  +----------------+
                  | Load Balancer  |
                  +----------------+
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
         Server 1      Server 2      Server 3
             |             |             |
             +-------------+-------------+
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
           Cache       Database      AI Services
```

The key idea for today is:

> **Load Balancer distributes traffic → multiple servers provide scale → health checks remove failed servers → auto scaling adjusts capacity → stateless servers make horizontal scaling easier.**

**Next: Day 6 — Message Queues + Asynchronous Systems.** This is where you'll learn why we don't make every request wait for slow work, and it will become very important for **GenAI, RAG, document processing, and Agentic AI architectures**.
