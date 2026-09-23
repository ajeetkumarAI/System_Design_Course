# Day 3: Monolithic vs. Microservices Architecture

---

## 1. Executive Summary & Core Architectural Mental Model

When evaluating system architectures in FAANG/Tier-1 software engineering interviews, the fundamental trade-off is **Operational Simplicity & Direct Function Calls (Monolith)** versus **Autonomous Deployment & Scalability Boundaries (Microservices)**.

```
┌─────────────────────────────────────────┐      ┌─────────────────────────────────────────┐
│          MONOLITHIC ARCHITECTURE        │      │        MICROSERVICES ARCHITECTURE       │
│                                         │      │                                         │
│  ┌───────────────────────────────────┐  │      │  ┌──────────────┐     ┌──────────────┐  │
│  │ Single Application Runtime        │  │      │  │ Order Service│     │ Auth Service │  │
│  │ (In-Process Function Calls)       │  │      │  └──────┬───────┘     └──────┬───────┘  │
│  └─────────────────┬─────────────────┘  │      │         │ Network (gRPC/HTTP)│        │
│                    │                    │      │         ▼                    ▼        │
│                    ▼                    │      │  ┌──────────────┐     ┌──────────────┐  │
│  ┌───────────────────────────────────┐  │      │  │ Order Database│    │ Auth Database│  │
│  │ Single Centralized Database       │  │      │  └──────────────┘     └──────────────┘  │
│  └───────────────────────────────────┘  │      │                                         │
└─────────────────────────────────────────┘      └─────────────────────────────────────────┘
```

---

## 2. High-Level Comparison & Trade-off Matrix

| Criterion | Monolithic Architecture | Microservices Architecture |
| :--- | :--- | :--- |
| **Deployment Unit** | Single executable binary / JAR / container | Dozens or hundreds of independently deployed services |
| **Communication Mechanism**| In-memory function/method calls (nanoseconds) | Network protocols (gRPC, REST, Kafka) (milliseconds) |
| **Data Storage Model** | Single central database (Shared Schema) | Database-per-Service pattern (Polyglot Persistence) |
| **Data Consistency** | **ACID** (Strong Consistency via DB transactions) | **BASE** (Eventual Consistency via Sagas / CDC) |
| **Failure Domain / Blast Radius**| Unhandled crash down-scales/fails entire app | Single service failure isolated via Circuit Breakers |
| **Team Topology & Velocity**| Best for 1–20 engineers; high merge friction | Ideal for 50+ engineers across multiple autonomous squads |
| **Infrastructure Complexity**| Low (Standard App Server + DB) | High (Kubernetes, Service Mesh, Tracing, API Gateways) |

---

## 3. Deep Dive: Microservices Architecture Mechanics

In a microservices paradigm, the system is decomposed into small, loosely coupled, independently deployable services organized around **business capabilities** (Bounded Contexts in Domain-Driven Design).

### Key Architectural Patterns
1. **API Gateway Pattern:** Acts as the single entry point for all clients. Handles SSL termination, authentication, rate limiting, and request routing (e.g., Kong, AWS API Gateway, Envoy).
2. **Database-Per-Service:** Each microservice strictly owns its data store. Services cannot query each other's databases directly; all interaction occurs via API or events.
3. **Event-Driven Asynchronous Communication:** Services publish state changes as events to message brokers (e.g., Apache Kafka, RabbitMQ) to decouple write pipelines.
4. **Service Discovery:** Dynamic IP resolution for elastic instances using registries like Consul or Kubernetes CoreDNS.

---

## 4. Real-World Case Studies: Monolith to Microservices (And Back)

### Case Study A: Netflix (The Microservices Pioneer)
* **The Shift:** Transitioned from a monolithic database architecture to microservices between 2008 and 2016 following a major database outage.
* **Architecture:** Over **1,000+ microservices** handling client playback, recommendations, search, and encoding.
* **Key Lesson:** Utilized heavy resilience patterns (Hystrix circuit breakers, Chaos Engineering via Chaos Monkey) to handle cascading failures across distributed services.

### Case Study B: Amazon Prime Video (The "Microservices Back to Monolith" Pivot)
* **The Pivot:** Amazon Prime Video's Video Quality Analysis engine migrated from a distributed serverless microservices setup (AWS Step Functions + Lambda) back to a **monolithic process architecture**.
* **Why?** Passing massive video frame data across network/S3 boundaries between microservices created extreme latency and networking costs. Consolidation into a single monolithic process reduced infrastructure cost by **90%** and improved throughput significantly.
* **FAANG Interview Takeaway:** Microservices are **not** inherently superior. High-throughput data-intensive pipelines with tight couplings often belong in a unified process or modular monolith.

---

## 5. Handling Data Consistency across Microservices

In a monolith, ACID transactions guarantee $100\%$ consistency:
```sql
BEGIN TRANSACTION;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  UPDATE orders SET status = 'PAID' WHERE id = 50;
COMMIT;
```

In Microservices, cross-service ACID transactions are impossible due to network isolation. FAANG candidates must explain two primary patterns for handling distributed transactions:

### 1. The Saga Pattern (Sagas)
A sequence of local transactions. Each service updates its own database and publishes an event to trigger the next step.
* **Choreography-based Saga:** Services listen to events and decide actions autonomously without a central orchestrator.
* **Orchestration-based Saga:** A central coordinator service directs participant services on which local transactions to execute.
* **Compensating Transactions:** If step $N$ fails, the system executes explicit rollback operations for steps $1$ through $N-1$ (e.g., issuing a refund if order fulfillment fails).

### 2. Two-Phase Commit (2PC)
A protocol where a coordinator asks all distributed nodes to **Prepare** and then **Commit**.
* *Warning:* Avoid recommending 2PC in system design interviews for high-scale systems due to high locking overhead, blocking behavior, and latency spikes.

---

## 6. Detailed Decision Decision Tree for System Design Interviews

```
                        Is your engineering org > 50+ engineers?
                                     /            \
                                    NO            YES
                                   /                \
                       Use MODULAR MONOLITH       Do you need distinct technology stacks
                     (Optimizes dev speed)        or independent elasticity per component?
                                                      /            \
                                                     NO            YES
                                                    /                \
                                       Use MODULAR MONOLITH    Use MICROSERVICES
                                       (Focus on domain       (Enforce Service APIs,
                                         boundaries)          K8s, Event Bus)
```

---

## 7. FAANG System Design Interview Questions & Answers

### Q1: "When would you explicitly advise AGAINST microservices?"
**Answer:** 
1. **Early-stage products / startups (0 to 1 stage):** Domain boundaries are fluid. Refactoring network APIs across services takes $5\times$ longer than refactoring functions within a monolith.
2. **Small engineering team ($< 20$ engineers):** The operational overhead of Kubernetes, distributed tracing, and service meshes drains engineering capacity away from product features.
3. **Ultra-low latency data pipelines:** When network serialization overhead (JSON/gRPC) across service hops introduces intolerable latency bottlenecks (e.g., Amazon Prime Video framing engine).

### Q2: "How do you prevent cascading failures in a microservices network?"
**Answer:**
1. **Circuit Breakers:** Trip open when downstream error rates exceed a threshold (e.g., Resilience4j, Envoy) to fail fast and prevent thread starvation.
2. **Rate Limiting & Shedding:** Drop non-critical requests at the API Gateway during peak traffic surges.
3. **Timeouts & Retries with Exponential Backoff + Jitter:** Prevent thundering herd problems on recovering services.
4. **Bulkheading:** Isolate thread pools per downstream service so an outage in one service does not consume all execution threads.
