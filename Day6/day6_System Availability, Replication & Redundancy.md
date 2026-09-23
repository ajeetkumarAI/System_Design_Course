# Day 6: System Availability, Replication & Redundancy

---

## 1. Executive Summary & Definitions

In large-scale distributed systems, system availability measures the uptime and operational readiness of a service over a given period.

* **Availability:** The probability or percentage of time that a system remains operational and accessible to process requests when needed.
$$\text{Availability} = \frac{\text{Uptime}}{\text{Uptime} + \text{Downtime}} \times 100\%$$
* **Redundancy:** The practice of duplicating critical components or functions of a system to eliminate **Single Points of Failure (SPOF)**.
* **Replication:** A specific data-level redundancy pattern where copies of data are continuously synchronized across multiple physical servers or datacenters.

---

## 2. High Availability Metrics: SLA, SLO & "Nines"

FAANG companies define system availability using high-availability standards referred to as **"Nines of Availability"**:

| Availability Level | Downtime per Year | Downtime per Month | Typical Use Case |
| :--- | :--- | :--- | :--- |
| **99% ("Two Nines")** | 3.65 days | 7.31 hours | Internal tools, non-critical batch processors |
| **99.9% ("Three Nines")** | 8.76 hours | 43.8 minutes | Standard commercial SaaS APIs |
| **99.99% ("Four Nines")** | 52.6 minutes | 4.38 minutes | E-commerce checkout services, Authentication |
| **99.999% ("Five Nines")** | 5.26 minutes | 26.3 seconds | Financial payment gateways, Telecom infrastructure |

### Key Reliability Metrics:
* **MTBF (Mean Time Between Failures):** The average operational time between inherent system failures.
* **MTTR (Mean Time To Repair):** The average time required to repair a failed system and restore full functionality.
$$\text{Availability} = \frac{\text{MTBF}}{\text{MTBF} + \text{MTTR}}$$

---

## 3. Redundancy Architecture: Active-Passive vs. Active-Active

Redundancy ensures that if an active instance crashes, another node is ready to handle traffic immediately.

```
1. Active-Passive Topology:
┌──────────┐      Primary       ┌───────────────┐
│ Client   │ ─────────────────► │ Primary Node  │ ──(DB Sync)──┐
└──────────┘                    └───────────────┘              │
                                  (Fails over)                 ▼
                                ┌───────────────┐      ┌───────────────┐
                                │ Standby Node  │ ◄────┤ Standby DB    │
                                └───────────────┘      └───────────────┘

2. Active-Active Topology:
                  ┌───────────────┐
             ┌──► │ Active Node A │ ──┐
┌──────────┐ │    └───────────────┘   │    ┌───────────────┐
│ Load     │─┤                        ├──► │ Shared / Sync │
│ Balancer │ │    ┌───────────────┐   │    │ Data Layer    │
└──────────┘ └──► │ Active Node B │ ──┘    └───────────────┘
                  └───────────────┘
```

### Active-Passive (Primary-Standby)
* **How it works:** Traffic is directed strictly to the Primary node. The Secondary (Standby) node remains idle, keeping state in sync via heartbeats.
* **Failover Mechanism:** If the Primary fails, a failover mechanism (e.g., DNS switch, VIP migration via Keepalived) routes traffic to the Passive node.
* **Trade-offs:** Wasted compute capacity in idle state; slight failover latency during cold start / IP propagation.

### Active-Active
* **How it works:** All nodes actively handle real incoming user traffic concurrently behind a Load Balancer.
* **Trade-offs:** Maximum infrastructure utilization and zero-downtime failover; increased complexity in state synchronization and distributed concurrency handling.

---

## 4. Replication Strategies (Data-Level Redundancy)

Replication guarantees data availability across multiple database nodes if a physical disk or datacenter fails.

```
┌────────────────────────────────────────────────────────────────────────┐
│                        REPLICATION MODES                               │
├───────────────────────────────────┬────────────────────────────────────┤
│     Synchronous Replication       │     Asynchronous Replication       │
│                                   │                                    │
│  Client ──► Primary ──► Replica   │  Client ──► Primary (Ack instantly)│
│               │             │     │               │                    │
│               └─(Wait Ack)──┘     │               └──(Async Sync)───►  │
│                                   │                                    │
│ • Strong Consistency ($Zero$ data loss)│ • Ultra-low write latency          │
│ • High write latency              │ • Risk of data loss on crash       │
└───────────────────────────────────┴────────────────────────────────────┘
```

### 1. Single-Leader (Master-Slave / Primary-Replica)
* All **Writes** go to the Primary node.
* The Primary streams changelogs (WAL/binlog) to Replica nodes.
* Replicas serve **Read-only** queries to scale read throughput.

### 2. Multi-Leader (Active-Active Replication)
* Multiple database nodes accept write operations simultaneously.
* Essential for multi-region global deployments (e.g., AWS Aurora Global Database).
* Requires conflict resolution strategies (e.g., *Last-Write-Wins (LWW)*, *CRDTs*).

### 3. Leaderless Replication (Dynamo-Style)
* Popularized by Amazon DynamoDB and Apache Cassandra.
* Any replica accepts writes and reads using **Quorum Consensus**:
  $$R + W > N$$
  *(where $N$ = total replicas, $W$ = write quorum, $R$ = read quorum)*.

---

## 5. Comparison: Redundancy vs. Replication

| Feature | Redundancy | Replication |
| :--- | :--- | :--- |
| **Primary Scope** | Compute, Infrastructure, Network, Services | Data Storage (Databases, Cache, Files) |
| **Core Objective** | Eliminate Single Points of Failure (SPOF) | Prevent data loss & enable high read throughput |
| **Implementation** | Duplicate EC2 instances, Load Balancers, Power units | DB Master-Replica sync, Cross-region S3 replication |
| **Failure Handling** | Automatic failover / traffic rerouting | Replica promotion to Master (Failover) |

---

## 6. FAANG System Design Interview Questions & Answers

### Q1: "How do you achieve 99.999% ('Five Nines') availability in a distributed application?"
**Answer Framework:**
1. **Eliminate All Single Points of Failure (SPOF):** Redundant load balancers (Active-Active BGP routing), multi-AZ app deployments, and multi-region database replication.
2. **Automated Health Checks & Self-Healing:** Implement proactive health check probes to automatically drop degraded nodes from service discovery.
3. **Graceful Degradation & Circuit Breaking:** Fallback gracefully to cache layers or default states when secondary services fail.
4. **Zero-Downtime Deployments:** Use Blue-Green or Canary deployment pipelines to validate releases without service interruption.

### Q2: "What happens when the Primary node crashes in a Primary-Replica database setup?"
**Answer Framework:**
1. **Detection:** Sentinel/Consul nodes detect missed heartbeats from the Primary within a predefined timeout.
2. **Leader Election:** Replicas trigger a consensus election (e.g., Raft, Paxos) to choose the replica with the most up-to-date Write-Ahead Log (WAL).
3. **Promotion & Failover:** The elected replica is promoted to Primary (Read-Write mode), and dynamic DNS/connection pools update automatically.
4. **Split-Brain Mitigation:** Use fence tokens to prevent the old primary from continuing writes if it recovers from a temporary network partition.
