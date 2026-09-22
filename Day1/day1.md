# Lecture Notes: System Design Process in Software Engineering

---

## 1. Introduction to System Design

System Design is the process of defining the **architecture, components, modules, interfaces, and data** for a system to satisfy specified requirements.

- It translates business and user requirements into a functional, scalable, and reliable software architecture.
- Bridges the gap between **software requirements analysis** and **software implementation/coding**.

---

## 2. Key Objectives of System Design

1. **Scalability:** Ensure the system handles load growth (traffic, storage, concurrent users).
2. **Reliability & Availability:** Guarantee uptime and graceful failure handling.
3. **Maintainability:** Keep code and architecture modular for future enhancements.
4. **Efficiency:** Optimize resource usage (CPU, memory, bandwidth, latency).
5. **Security:** Protect data integrity and manage access control.

---

## 3. High-Level Design (HLD) vs. Low-Level Design (LLD)

```
                       ┌─────────────────────────┐
                       │   System Requirements   │
                       └────────────┬────────────┘
                                    │
                                    ▼
                       ┌─────────────────────────┐
                       │ High-Level Design (HLD) │
                       │ (System Architecture)   │
                       └────────────┬────────────┘
                                    │
                                    ▼
                       ┌─────────────────────────┐
                       │ Low-Level Design (LLD)  │
                       │ (Detailed Component)    │
                       └─────────────────────────┘
```

| Aspect | High-Level Design (HLD) | Low-Level Design (LLD) |
| :--- | :--- | :--- |
| **Focus** | System-wide architecture and macro view | Detailed component design and micro view |
| **Target Audience** | System Architects, Tech Leads, Stakeholders | Developers and Software Engineers |
| **Key Output** | Diagrams (Database schemas, API protocols, Network topology) | Class diagrams, sequence diagrams, pseudo-code |
| **Scope** | Monolith vs. Microservices, DB choices, Load Balancers | Data structures, algorithms, interface signatures |

---

## 4. The System Design Process (Step-by-Step)

```
┌──────────────────────────────────────────────────────────┐
│ Step 1: Requirement Clarification (Functional & Non-Func) │
└────────────────────────────┬─────────────────────────────┘
                             │
                             ▼
┌──────────────────────────────────────────────────────────┐
│ Step 2: System Interface & API Definition                │
└────────────────────────────┬─────────────────────────────┘
                             │
                             ▼
┌──────────────────────────────────────────────────────────┐
│ Step 3: Back-of-the-Envelope Estimation                  │
└────────────────────────────┬─────────────────────────────┘
                             │
                             ▼
┌──────────────────────────────────────────────────────────┐
│ Step 4: Data Model & Database Design                     │
└────────────────────────────┬─────────────────────────────┘
                             │
                             ▼
┌──────────────────────────────────────────────────────────┐
│ Step 5: High-Level Architecture Design                   │
└────────────────────────────┬─────────────────────────────┘
                             │
                             ▼
┌──────────────────────────────────────────────────────────┐
│ Step 6: Detailed Component Design & Bottlenecks          │
└────────────────────────────┴─────────────────────────────┘
```

### Step 1: Requirement Clarification
- **Functional Requirements:** Features the system must perform (e.g., *User can post a tweet*).
- **Non-Functional Requirements:** Quality attributes and constraints (e.g., *99.99% availability, latency < 200ms*).

### Step 2: API & Interface Design
- Define endpoints (REST, gRPC, GraphQL) and core parameters passed between client and server.

### Step 3: Capacity Estimation (Back-of-the-Envelope)
- Calculate expected traffic (QPS, TPS), storage bandwidth, and memory/cache requirements.

### Step 4: Data Model & Storage Choice
- Choose appropriate data stores based on read/write patterns:
  - **SQL (Relational):** ACID compliance, structured data (e.g., PostgreSQL, MySQL).
  - **NoSQL:** High horizontal scalability, key-value / document / graph (e.g., MongoDB, Cassandra).

### Step 5: High-Level System Architecture
- Sketch core components:
  - **Load Balancers:** Distribute traffic evenly across servers.
  - **Application Servers:** Process business logic.
  - **Caches:** Reduce database read latency (e.g., Redis, Memcached).
  - **Message Queues:** Asynchronous processing (e.g., Kafka, RabbitMQ).

### Step 6: Identifying Bottlenecks & Optimization
- Address single points of failure (SPOF), replication, sharding, and fault tolerance mechanisms.

---

## 5. Core System Design Concepts

- **Load Balancing:** Algorithms like Round Robin, Least Connections, and Consistent Hashing.
- **Caching Strategies:** Cache-aside, Read-through, Write-through, Write-back.
- **Database Sharding & Replication:** Horizontal partitioning and Leader-Follower setups.
- **CAP Theorem:** Consistency, Availability, Partition Tolerance (choose two).

---

*This lecture summary is compiled based on the video: [What is system design process in software engineering ?](https://www.youtube.com/watch?v=43-X22tdxiI).*
