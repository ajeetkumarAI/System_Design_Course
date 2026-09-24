# What is Consistency in System Design? (Strong vs. Eventual Consistency)

This guide summarizes the key concepts from the video on **Data Consistency** in Distributed Systems, tailored for System Design Interviews (HLD).

---

## 1. Core Concept: What is Consistency?

In a distributed system, data is replicated across multiple servers or databases to ensure **high availability** and **fault tolerance**. 

**Consistency** refers to the state where all nodes/replicas in a distributed cluster see and return the **same data at the exact same time**.

When a write operation updates a value in a database, consistency dictates *when* and *how* that updated value becomes visible to read operations across all nodes.

```
       [ Client / API ]
          /        \
   (Write)          (Read)
        /            \
       v              v
  [ Node A ]  <---> [ Node B ]
   (Primary)  Replication (Replica)
```

---

## 2. Types of Data Consistency

### A. Strong Consistency
* **Definition:** Guarantees that as soon as a write operation completes, any subsequent read operation across any node in the system will immediately return the latest updated value.
* **Mechanism:** 
  1. Client sends a write request to Node A.
  2. Node A updates its state and synchronously replicates the update to Node B, Node C, etc.
  3. Node A waits for acknowledgments from all replicas before confirming success to the client.
* **Trade-off:** High consistency, but **higher latency** and **lower availability** (if a node is unreachable or network partitions occur).
* **Use Cases:**
  * Banking & Financial transactions (Account Balance)
  * Inventory management during flash sales
  * Reservation systems (Flight/Hotel booking)

---

### B. Eventual Consistency
* **Definition:** Guarantees that if no new updates are made, all replicas will eventually converge and return the same, latest value. However, there may be a temporary delay (replication lag) where reads return stale data.
* **Mechanism:**
  1. Client writes data to Node A.
  2. Node A immediately acknowledges success to the client.
  3. Node A asynchronously replicates the data to Node B and Node C in the background.
* **Trade-off:** **Low latency** and **high availability**, but temporary data inconsistency across nodes.
* **Use Cases:**
  * Social media feeds (Post likes, comment counts)
  * User profile updates / Avatars
  * YouTube view counts / Video recommendations

---

## 3. Comparison Matrix

| Feature | Strong Consistency | Eventual Consistency |
| :--- | :--- | :--- |
| **Data Freshness** | Immediate guarantee | Delayed (eventual) |
| **Latency** | Higher (Synchronous writes) | Lower (Asynchronous writes) |
| **Availability** | Lower (CAP Theorem constraint) | Higher |
| **Complexity** | High write lock/coordination overhead | High conflict-resolution overhead |
| **Typical Databases** | RDBMS (PostgreSQL, MySQL), Spanner | NoSQL (Cassandra, DynamoDB) |

---

## 4. Key Takeaways for System Design Interviews

1. **CAP Theorem Relation:**
   * **CP Systems (Consistency + Partition Tolerance):** Choose Strong Consistency over availability when partition occurs.
   * **AP Systems (Availability + Partition Tolerance):** Choose Eventual Consistency to maintain high availability and performance.

2. **How to Choose in an Interview:**
   * Always identify the business context:
     * Does reading stale data cause business loss/legal issues? $\rightarrow$ **Strong Consistency**.
     * Is high throughput and low user-perceived latency more critical than immediate accuracy? $\rightarrow$ **Eventual Consistency**.
