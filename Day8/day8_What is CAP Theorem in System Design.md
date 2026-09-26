# What is CAP Theorem in System Design?

This guide summarizes the key concepts from the video **"What is CAP theorem in Hindi?"** by Engineering Digest, tailored for System Design Interviews (HLD).

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

## 5. Key System Design Interview Takeaways

1. **You cannot choose "CA" in distributed systems:** A "CA system" implies no network partitions exist, which only applies to single-node monolithic databases.
2. **PACELC Theorem Extension:** 
   * If there is a Partition (**P**), choose between Availability (**A**) and Consistency (**C**).
   * **Else (E)** (when the system is running normally), choose between Latency (**L**) and Consistency (**C**).
3. **Decision Rule:** Ask whether reading outdated data causes severe business issues (e.g., spending the same money twice $\rightarrow$ **CP**) or if slight lag is acceptable (e.g., YouTube view counter $\rightarrow$ **AP**).
