# Day 5: System Throughput & Performance Optimization (FAANG Deep Dive)

---

## 1. Core Definition: What is System Throughput?

In distributed systems and software engineering, **Throughput** measures the volume of work or data a system can successfully process within a given timeframe.

### Key Units of Measurement:
* **RPS / QPS:** Requests Per Second / Queries Per Second (e.g., API servers, database reads).
* **TPS:** Transactions Per Second (e.g., payment gateways, database writes).
* **Data Rate:** Megabytes/Gigabytes per second ($\text{MB/s}$, $\text{GB/s}$) or Bits per second ($\text{Gbps}$) (e.g., streaming pipelines, network interfaces).

$$\text{Throughput} = \frac{\text{Total Processed Units (Requests / Bytes)}}{\text{Total Time Elapsed}}$$

---

## 2. Throughput vs. Latency vs. Bandwidth

Understanding how throughput relates to other performance metrics is essential for system design interviews:

| Metric | Definition | Analogy (Highway Traffic) |
| :--- | :--- | :--- |
| **Latency** | Time taken for a single request to complete ($\text{ms}$) | The time it takes for one car to travel from point A to B |
| **Throughput** | Number of requests completed per unit time ($\text{RPS}$) | The total number of cars passing a toll bridge per minute |
| **Bandwidth** | Maximum theoretical capacity of the network/pipe | The total number of lanes on the highway |

> **Little's Law:** In a stable system, the average number of concurrent requests $L$ equals long-term average effective arrival rate $\lambda$ multiplied by the average response time $W$:
> $$L = \lambda \times W \quad \implies \quad \text{Throughput } (\lambda) = \frac{\text{Concurrency } (L)}{\text{Latency } (W)}$$

---

## 3. Real-World Analogy & Concrete Examples

### Analogy: The Restaurant Kitchen
* **High Latency / Low Throughput:** A single chef cooks one order from start to finish (15 mins per order). Maximum throughput = 4 orders/hour.
* **Low Latency / High Throughput:** Multiple specialized chefs work on an assembly line with parallel preparation. Throughput = 60 orders/hour.

### Software Concrete Example:
Consider an API endpoint processing user profile updates:
* **Baseline System:**
  * Average request processing time (Latency) = $100\text{ ms}$ ($0.1\text{ s}$).
  * Single-threaded server capacity = $\frac{1}{0.1\text{ s}} = 10\text{ RPS}$.
* **Optimized System (Concurrency & Non-blocking I/O):**
  * Using asynchronous worker threads ($100$ concurrent connections handling non-blocking DB calls).
  * Effective System Throughput = $100 \times 10\text{ RPS} = 1,000\text{ RPS}$.

---

## 4. Key Bottlenecks Limiting System Throughput

```
┌────────────────────────────────────────────────────────────────────────┐
│                        THROUGHPUT BOTTLENECKS                          │
├───────────────────┬───────────────────┬────────────────────────────────┤
│   CPU-Bound       │    I/O-Bound      │       Concurrency/Locks        │
│                   │                   │                                │
│ • Heavy Compute   │ • Slow DB Queries │ • Mutex Lock Contention        │
│ • Serialization   │ • Disk Read/Write │ • Database Connection Pools    │
│ • Compression     │ • Network Delays  │ • Thread Context Switching     │
└───────────────────┴───────────────────┴────────────────────────────────┘
```

1. **Database Disk I/O:** Unindexed database queries causing full table scans.
2. **Synchronous Network Calls:** Sequential HTTP calls to downstream services ("Waterfall requests").
3. **Thread Starvation & Lock Contention:** Thread pools getting blocked waiting for shared resources/locks.
4. **Memory Swapping & Garbage Collection:** GC pauses blocking the event loop or execution threads.

---

## 5. Practical Strategies to Increase Throughput

To improve system throughput from hundreds to hundreds-of-thousands of QPS, engineers utilize the following architectural patterns:

### Strategy 1: Asynchronous & Non-Blocking I/O
* **Concept:** Avoid blocking execution threads while waiting for network or disk I/O.
* **Implementation:** Event loops (Node.js, Netty, Tokio), Async/Await, or Reactive frameworks (RxJava).

### Strategy 2: Horizontal Scaling & Load Balancing
* **Concept:** Distribute incoming traffic across multiple stateless application instances.
* **Implementation:** Use Layer 4 (TCP) or Layer 7 (HTTP) load balancers (e.g., NGINX, HAProxy, AWS ALB) with Round-Robin or Least Connections algorithms.

```
                   ┌──────────────┐
                   │ Load Balancer│
                   └──────┬───────┘
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
   ┌────────────┐  ┌────────────┐  ┌────────────┐
   │ App Node 1 │  │ App Node 2 │  │ App Node 3 │
   └────────────┘  └────────────┘  └────────────┘
```

### Strategy 3: Caching (In-Memory & Edge)
* **Concept:** Serve frequent reads directly from memory to prevent expensive database roundtrips.
* **Implementation:** Redis/Memcached cluster for dynamic data; CDNs for static assets.

### Strategy 4: Batching & Bulk Operations
* **Concept:** Group multiple small requests into a single batch to reduce network roundtrip overhead and disk flush operations.
* **Implementation:** Kafka batch producers, SQL batch inserts (`INSERT INTO ... VALUES (...), (...)`).

### Strategy 5: Database Optimization (Read Replicas & Sharding)
* **Read-Heavy Workloads:** Offload read queries to read-replicas.
* **Write-Heavy Workloads:** Horizontal partitioning (sharding) by Hash or Range key.

---

## 6. FAANG System Design Interview Questions & Answers

### Q1: "You have an API server handling 1,000 RPS, but the system needs to scale to 50,000 RPS. How do you approach this throughput bottleneck?"
**Answer Framework:**
1. **Identify the Bottleneck:** Analyze CPU usage, Memory, Network I/O, and DB metrics (Slow queries, Connection pool saturation).
2. **Scale Stateless Layer:** Place app servers behind a Layer 7 Load Balancer and scale out horizontally using Kubernetes auto-scaling (HPA).
3. **Decouple Heavy Logic:** Move long-running tasks (e.g., PDF generation, video processing) to asynchronous message queues (Kafka / RabbitMQ) with background worker pools.
4. **Optimize Read/Write Paths:**
   * Introduce Redis for hot reads.
   * Add database read replicas.
   * Implement connection pooling (e.g., PgBouncer) to reduce connection handshake overhead.

### Q2: "Why can decreasing latency increase throughput, but increasing throughput might increase latency?"
**Answer:**
* **Decreasing Latency $\rightarrow$ Increases Throughput:** Shorter processing time per request frees up worker threads faster, allowing the system to handle more requests per second ($\text{Throughput} = \frac{\text{Concurrency}}{\text{Latency}}$).
* **Increasing Throughput $\rightarrow$ Might Increase Latency:** Driving throughput near peak system capacity causes queue buildup (router buffers, thread pools, DB wait queues). Requests spend more time waiting in queues before processing, leading to higher tail latency ($P99$ latency spikes).
