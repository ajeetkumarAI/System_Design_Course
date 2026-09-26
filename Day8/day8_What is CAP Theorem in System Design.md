# What is CAP Theorem in System Design?

---

## 1. What is CAP Theorem?

The **CAP Theorem** (also known as Brewer's Theorem) states that in any distributed data store, you can only guarantee **at most two out of three** of the following properties at the same time:

1. **C** – Consistency
2. **A** – Availability
3. **P** – Partition Tolerance

In a distributed system connected over a network, **Network Partitions (P) are unavoidable**. Therefore, when a network failure occurs, system designers must choose between **Consistency (C)** and **Availability (A)**.

```
                  CAP Theorem
                    /  |  \
                   /   |   \
     Consistency  /    |    \  Availability
        (C) -----/-----|-----\----- (A)
                \      |      /
                 \     |     /
                  Partition
                Tolerance (P)
```

---

## 2. Breaking Down C, A, and P

### A. Consistency (C)
* **Definition:** Every read receives the most recent write or an error. All nodes in the cluster see the exact same data at the exact same time.
* **How it works:** When a write operation happens on Node A, the system locks or delays response until the update is replicated to Node B and Node C.
* **Example:** Financial transactions, bank account balances.

### B. Availability (A)
* **Definition:** Every non-failing node returns a non-error response for every request, but without the guarantee that it contains the most recent write.
* **How it works:** Even if network communication between Node A and Node B breaks, Node B will still respond to client requests using whatever data it currently has (even if stale).
* **Example:** Social media posts/likes, e-commerce product listings.

### C. Partition Tolerance (P)
* **Definition:** The system continues to operate despite an arbitrary number of messages being dropped or delayed by the network between nodes (a **Network Partition**).
* **Reality check:** Distributed systems rely on physical networks, cables, and routers, which can fail. Thus, **Partition Tolerance (P) is a mandatory requirement**, not an option, in modern distributed systems.

---

## 3. The Trade-Off: CP vs. AP Systems

Because **P (Partition Tolerance)** is non-negotiable in distributed networks, the choice comes down to **CP** or **AP**:

```
                  Network Partition Occurs!
                            /   \
                           /     \
                          v       v
                     CP System   AP System
```

### A. CP Systems (Consistency + Partition Tolerance)
* **Behavior:** When a network partition occurs, the system refuses or delays writes/reads on isolated nodes to maintain strict data consistency across reachable nodes.
* **Trade-off:** High consistency, but **reduced availability** during network failures.
* **Databases:** MongoDB, HBase, Redis (in cluster mode), PostgreSQL / MySQL (with synchronous replication).

### B. AP Systems (Availability + Partition Tolerance)
* **Behavior:** When a network partition occurs, both sides of the partition remain active and continue answering queries. Replicas temporarily become out-of-sync, but system uptime is prioritized.
* **Trade-off:** High availability and low latency, but risks returning **stale data** (Eventual Consistency).
* **Databases:** Apache Cassandra, Amazon DynamoDB, CouchDB.

---

## 4. Summary Matrix

| System Type | Focus | Primary Advantage | Trade-Off | Common Uses |
| :--- | :--- | :--- | :--- | :--- |
| **CP** | Consistency & Partition Tolerance | Always accurate data | Writes/reads fail during network breaks | Banking, Payments, Inventories |
| **AP** | Availability & Partition Tolerance | High uptime, fast responses | Temporary stale data (Eventual Consistency) | Social Feeds, Like Counts, Comments |

---

# CAP Theorem System Classification: 50 Real-World Examples

The **CAP Theorem** dictates that a distributed system running over an unreliable network (which guarantees Network Partitions, **P**) must choose between **Consistency (CP)** or **Availability (AP)**:

* **CP Systems (Consistency & Partition Tolerance):** Prioritize data accuracy and atomic correctness. If a partition occurs, the system refuses writes or delays reads to prevent stale or conflicting data.
* **AP Systems (Availability & Partition Tolerance):** Prioritize system uptime and low-latency responses. The system remains accessible during network breaks, serving cached or temporarily stale data that converges later (Eventual Consistency).

---

## High-Level Summary Matrix

| System Type | Focus | Primary Advantage | Trade-Off | Common Uses |
| :--- | :--- | :--- | :--- | :--- |
| **CP** | Consistency & Partition Tolerance | Always accurate, atomic, and fresh data. | Writes/reads fail or block during network partitions. | Banking, Ledger Payments, Real-time Inventories, Seat Reservations |
| **AP** | Availability & Partition Tolerance | High uptime, zero downtime, fast global responses. | Temporary stale data across nodes (Eventual Consistency). | Social Feeds, Like/View Counts, Streaming, Telemetry, Caching |

---

## 25 CP Real-World Examples (Consistency & Partition Tolerance)

These systems prioritize **correctness over uptime**. Returning an error or temporarily going offline is preferable to serving incorrect, duplicate, or stale data.

| # | System / Microservice | Primary Reason & Business Justification |
| :---: | :--- | :--- |
| **1** | **Bank Account Balance Engine** | Ledger balances cannot suffer dirty reads; debiting an account must immediately reflect everywhere. |
| **2** | **ATM Cash Dispenser Service** | Must verify and lock account balance centrally before physically dispensing currency notes. |
| **3** | **Movie Theater Seat Booking (BookMyShow/AMC)** | Enforces strict atomic locks to prevent two users from reserving the exact same physical seat. |
| **4** | **Airline Reservation System (Sabre/Amadeus)** | Overbooking seat allocations during network failures leads to legal and operational liabilities. |
| **5** | **Stock Exchange Order Matching Engine (NASDAQ/NSE)** | Execution order and stock valuation pricing must be linearizable and exact. |
| **6** | **Cryptocurrency Consensus Node (Bitcoin/Ethereum)** | Double-spending prevention requires absolute proof-of-work/proof-of-stake consensus. |
| **7** | **Payment Gateway Integration (Stripe/PayPal)** | Idempotency and transaction processing must be consistent to avoid duplicate card charges. |
| **8** | **E-Commerce Inventory Quantity Tracker** | Stock counts must lock during checkout so an item with `quantity = 1` isn't sold twice. |
| **9** | **Distributed Lock Manager (Redis Redlock/ZooKeeper)** | Cluster coordination primitives must guarantee strict mutual exclusion for critical sections. |
| **10** | **Rideshare Driver Matching Engine (Uber/Lyft)** | A driver cannot be simultaneously assigned to two separate pickup requests. |
| **11** | **User Authentication & Session Invalidation** | Password changes and revoked JWT/OAuth tokens must instantly block unauthorized access globally. |
| **12** | **Concert Ticket Queue System (Ticketmaster)** | High-concurrency ticket drops require deterministic FIFO queue reservation locks. |
| **13** | **Hospital Electronic Health Record (EHR)** | Patient medical histories, drug dosages, and allergies require exact data accuracy across nodes. |
| **14** | **Hotel Room Reservation Engine** | Prevents double-booking same hotel suite during overlapping date ranges. |
| **15** | **Digital Wallet Stored Value (Apple Pay Wallet/Paytm)** | Stored cash value transfers require ACID transactions across balance partitions. |
| **16** | **Government National ID Registry (Aadhaar/SSN)** | Citizen identifier creation must guarantee absolute uniqueness constraints across region nodes. |
| **17** | **Parking Spot Reservation Service** | Physical barrier gates and spot allocation rely on real-time binary occupancy state. |
| **18** | **High-Frequency Algorithmic Trading Bot** | Trade triggers rely on exact pricing feeds; stale price inputs lead to immediate financial loss. |
| **19** | **Cluster Leader Election Service (etcd in Kubernetes)** | Only one master controller node can hold lease lock at any given moment. |
| **20** | **Food Delivery Order Payment Checkout** | Cart value validation, promo code applicability, and wallet deduction must be atomic. |
| **21** | **Database Auto-Increment ID Service (Snowflake/Sequencer)** | Primary key generation must guarantee strict monotonicity and zero duplicate IDs. |
| **22** | **Domain Name Registrar (GoDaddy/Namecheap)** | Domain purchase must instantly claim global ownership in registry to stop domain squatting. |
| **23** | **Online Casino/Gambling Engine** | Game state, chip balances, and random number outcomes require exact auditability and locking. |
| **24** | **EV Charging Station Slot Reservation** | Physical charging plug availability must be updated instantly to avoid charger collision. |
| **25** | **Cloud Security IAM Policy Evaluator** | Privilege escalation revocation must apply instantly to block malicious requests. |

---

## 25 AP Real-World Examples (Availability & Partition Tolerance)

These systems prioritize **responsiveness and continuous operation**. Serving slightly outdated data or reconciling concurrent edits later is acceptable to ensure zero downtime.

| # | System / Microservice | Primary Reason & Business Justification |
| :---: | :--- | :--- |
| **1** | **YouTube Video View Counter** | Showing 1,000,000 vs 1,000,050 views is imperceptible to users; speed and global availability matter more. |
| **2** | **Twitter/X Timeline Feed Generation** | Users prefer seeing slightly delayed tweets over an error page saying "Feed Unavailable". |
| **3** | **Instagram Post Like Aggregator** | Highly distributed counters converge asynchronously via CRDTs or background batch sync. |
| **4** | **Netflix Home Screen Recommendations** | Recommending a movie watched 5 minutes ago on another device is better than blocking app load. |
| **5** | **Spotify Music Streaming Service** | Songs must play instantly without waiting for playback metrics to sync across global data centers. |
| **6** | **Google Maps Real-Time Traffic Layer** | Congestion heatmaps update continuously via crowd-sourced GPS data; lost pings are tolerable. |
| **7** | **E-Commerce Product Review Section** | A new review appearing 10 seconds late on another continent does not impact user experience. |
| **8** | **DNS (Domain Name System) Caching Hierarchy** | TTL-backed propagation allows stale IP resolution temporarily to keep the global internet fast. |
| **9** | **IoT Weather Sensor Data Ingestion** | Telemetry collectors log ambient temperature readings; dropped packets do not corrupt state. |
| **10** | **WhatsApp User "Last Seen" / Online Status** | Status updates are best-effort, broadcast asynchronously without strict read locks. |
| **11** | **Global Content Delivery Network (Cloudflare/Akamai)** | Edge nodes serve stale cached static assets if origin servers are unreachable. |
| **12** | **E-Commerce Shopping Cart (Amazon DynamoDB Model)** | Carts use conflict-resolution (e.g., merging cart items) so a customer can always add items. |
| **13** | **Uber Driver Location Map Tracking** | Smooth visual movement on map uses interpolated location pings; exact millisecond sync isn't required. |
| **14** | **Social Media Comment Section (Facebook/Reddit)** | Comments stream asynchronously; order conflicts are resolved via eventually consistent timestamps. |
| **15** | **Ad Impression Counter Engine** | Advertisers rely on aggregated batch billing reports rather than real-time synchronous locks. |
| **16** | **Fitness Tracker Daily Step Counter (Strava/Garmin)** | Steps synced offline reconcile asynchronously with cloud servers via multi-master sync. |
| **17** | **News Aggregator Homepage (Hacker News/BBC)** | Frontpage article rankings recalculate periodically on background cron jobs. |
| **18** | **Application Monitoring Metrics (Datadog/Prometheus)** | System CPU/RAM telemetry metrics value high-ingestion throughput over exact transaction locks. |
| **19** | **Twitch Live Stream Chat Room** | High-velocity messages drop or out-of-order deliver under extreme load to maintain live playback. |
| **20** | **E-Commerce Product Search Auto-Suggest** | Autocomplete dropdowns serve cached dictionary prefixes to maximize typing response speed. |
| **21** | **Collaborative Text Editing Offline Buffer (Figma/Notion)** | Local edits accumulate offline and merge automatically via Operational Transformation (OT/CRDT). |
| **22** | **Notification Dispatch Service (Firebase Push/APNS)** | Push alerts prioritize deliverability and low latency over strict sequential execution. |
| **23** | **Spam Filter Scoring Engine** | Email spam classification models operate on eventual updates of IP reputation blacklists. |
| **24** | **Web Crawler Indexer (Googlebot)** | Page indexing queue scales across millions of sites asynchronously without locking targets. |
| **25** | **Online Multiplayer Game Player Position (UDP)** | Network packets send continuously; lost frames drop cleanly rather than pausing gameplay. |

---

## 5. Key System Design Interview Takeaways

1. **You cannot choose "CA" in distributed systems:** A "CA system" implies no network partitions exist, which only applies to single-node monolithic databases.
2. **PACELC Theorem Extension:** 
   * If there is a Partition (**P**), choose between Availability (**A**) and Consistency (**C**).
   * **Else (E)** (when the system is running normally), choose between Latency (**L**) and Consistency (**C**).
3. **Decision Rule:** Ask whether reading outdated data causes severe business issues (e.g., spending the same money twice $\rightarrow$ **CP**) or if slight lag is acceptable (e.g., YouTube view counter $\rightarrow$ **AP**).
