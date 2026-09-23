# Day 4: Network Latency, CDN, and Caching Strategies (FAANG Deep Dive)

---

## 1. Core Definitions: Latency vs. Throughput

In distributed systems and FAANG system design interviews, performance optimization revolves around two critical metrics:

* **Latency:** The total time it takes for a request to travel from the client to the server and for the response to return to the client (measured in milliseconds, $\text{ms}$).
  $$\text{Latency} = \text{Propagation Delay} + \text{Transmission Delay} + \text{Queuing Delay} + \text{Processing Delay}$$
* **Throughput:** The amount of data successfully transmitted or processed per unit of time (measured in Requests Per Second (QPS) or Megabytes per second ($\text{MB/s}$)).

---

## 2. The Anatomy of Network Latency

Understanding where time is spent during a network request is vital for optimization:

```
┌──────────┐                                                             ┌──────────┐
│  Client  │ ──(DNS Lookup & TCP Handshake)────────────────────────────► │  Server  │
└────┬─────┘                                                             └────┬─────┘
     │                                                                        │
     │ 1. Propagation Delay (Speed of light in fiber optic cable)             │
     ├──────────────────────────────────────────────────────────────────────► │
     │                                                                        │
     │ 2. Transmission Delay ($Size \div Bandwidth$)                          │
     ├──────────────────────────────────────────────────────────────────────► │
     │                                                                        │
     │ 3. Queuing Delay (Router / Load balancer buffer wait times)             │
     ├──────────────────────────────────────────────────────────────────────► │
     │                                                                        │
     │ 4. Processing Delay (Application logic, DB query execution time)       │
     └──────────────────────────────────────────────────────────────────────► │
```

### Key Latency Numbers Every Engineer Should Know
* **L1 Cache Reference:** $0.5$ ns
* **Main Memory (RAM) Read:** $100$ ns
* **Round Trip within same Datacenter:** $\approx 0.5 - 1$ ms
* **Cross-continent Round Trip (e.g., California to New York):** $\approx 40 - 70$ ms
* **Satellite Internet Round Trip:** $\approx 500+$ ms

---

## 3. Caching Architecture & Strategies

Caching is the most effective technique to reduce database read load and latency by storing hot data in ultra-fast memory layers (e.g., Redis, Memcached).

### Caching Topologies
1. **In-Process (L1 Cache):** Stored inside the application server's local RAM (e.g., Guava, Caffeine, Node.js memory map). Zero network overhead, but prone to stale data and memory bloat across multiple server instances.
2. **Distributed Cache (L2 Cache):** Centralized cluster of cache nodes accessed via network (e.g., Redis Cluster, Memcached). Shared across all application servers.

### Cache Write Patterns
* **Cache-Aside (Lazy Loading):**
  * *Flow:* App checks cache $\rightarrow$ if miss, queries DB $\rightarrow$ populates cache $\rightarrow$ returns response.
  * *Pros:* Only requested data is cached; resilient if cache cluster fails.
  * *Cons:* Cache miss penalty (latency spike on first read); potential stale data if DB updates without cache invalidation.
* **Write-Through:**
  * *Flow:* App writes data directly to cache $\rightarrow$ cache synchronously writes to the database.
  * *Pros:* Data in cache is never stale.
  * *Cons:* Higher write latency because every write hits both cache and DB.
* **Write-Back (Write-Behind):**
  * *Flow:* App writes data to cache $\rightarrow$ cache asynchronously flushes writes to the database in batches.
  * *Pros:* Blazing fast write performance.
  * *Cons:* Risk of data loss if the cache node crashes before flushing to disk.

---

## 4. Cache Eviction Policies (When Memory is Full)

When cache memory fills up, eviction algorithms determine which keys to remove:

* **LRU (Least Recently Used):** Evicts items that haven't been accessed for the longest time. (Most common default in Redis).
* **LFU (Least Frequently Used):** Evicts items with the lowest access count frequency.
* **FIFO (First In, First Out):** Evicts the oldest added items regardless of access patterns.
* **TTL (Time-To-Live):** Expiration-based eviction where items automatically drop after a specified duration.

---

## 5. Content Delivery Networks (CDN) Architecture

A **CDN** is a globally distributed network of proxy servers (Points of Presence - PoPs) that cache static and dynamic content closer to end-users to minimize propagation delay.

```
┌──────────┐                                                 ┌──────────────────┐
│  Client  │ ──(Anycast DNS routes to nearest CDN PoP)──────► │ CDN Edge Server  │
└──────────┘                                                 └────────┬─────────┘
                                                                      │
                                                        [Cache Hit]   │ [Cache Miss]
                                                      Returns instantly ▼
                                                              ┌──────────────────┐
                                                              │ Origin Server    │
                                                              │ (AWS S3 / DC)    │
                                                              └──────────────────┘
```

### How CDN Works:
1. **Anycast Routing:** Directs client DNS queries to the geographically closest CDN PoP.
2. **Edge Caching:** Static assets (images, JS, CSS, video segments) are stored at the edge. If requested, the edge server returns them instantly ($\text{latency} < 10$ ms).
3. **Origin Shielding / Pull:** On a cache miss, the edge server requests data from the origin server once, caches it, and serves future regional requests.

---

## 6. CDN vs. Local Caching: Comparison Matrix

| Feature | CDN (Content Delivery Network) | Local / Distributed Caching (Redis/Memcached) |
| :--- | :--- | :--- |
| **Primary Location** | Global Edge servers (thousands of locations worldwide) | Datacenter or regional server clusters |
| **Target Data** | Static assets (HTML, images, videos, scripts) | Dynamic query results, user sessions, object models |
| **Latency Reduction** | Massive reduction in propagation delay (brings data to user's city) | Reduces database query and compute processing time |
| **Invalidation Control** | Purge via API / TTL expiration | Programmatic invalidation, pub/sub, TTL |

---

## 7. Advanced Techniques to Reduce Network Latency

1. **HTTP/2 & HTTP/3 (QUIC):**
   * HTTP/2 introduces multiplexing over a single TCP connection, eliminating head-of-line blocking.
   * HTTP/3 replaces TCP with **UDP (QUIC)**, eliminating TCP handshake and TLS negotiation latency during connection establishment.
2. **Connection Pooling & Keep-Alive:** Reuse existing TCP/gRPC connections instead of paying the handshake penalty for every request.
3. **Compression:** Use Gzip, Brotli, or WebP/AVIF formats to minimize transmission delay over limited network bandwidth.
4. **Geo-Routing & Multi-Region Deployment:** Deploy application replicas across multiple AWS/GCP regions (e.g., `us-east`, `eu-west`, `ap-south`) so users hit the nearest regional cluster.

---

## 8. FAANG System Design Interview Questions & Answers

### Q1: "How would you design a caching strategy for a viral video platform like TikTok or YouTube?"
**Answer Strategy:**
1. **Multi-Tiered Caching:** 
   * *Edge CDN:* Cache popular video chunks (`.ts` segments) at global CDN edge nodes near users.
   * *Origin Cache (Redis):* Cache video metadata, creator profiles, and engagement counters.
2. **Cache-Aside with TTL & LRU:** Hot videos remain in cache; cold videos naturally evict via LRU.
3. **Thundering Herd Mitigation:** When a viral video expires from cache, use **Distributed Locks** or **Probabilistic Early Expiration (XFetch)** so only one worker queries the database to rebuild the cache while others wait or serve stale data gracefully.

### Q2: "What is Cache Stampede (Thundering Herd) and how do you prevent it?"
**Answer Strategy:**
* *The Problem:* When a heavily requested cache key expires simultaneously, thousands of concurrent requests miss the cache and hammer the primary database all at once, causing cascading failure.
* *Solutions:*
  1. **Mutex / Distributed Locks:** Allow only one thread to regenerate the cache while others sleep or return stale data.
  2. **Asynchronous Proactive Refresh:** Background workers refresh keys before their TTL actually expires.
