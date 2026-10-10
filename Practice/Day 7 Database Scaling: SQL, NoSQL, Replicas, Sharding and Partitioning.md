# Day 7 — Database Scaling: SQL, NoSQL, Replicas, Sharding and Partitioning

Today, we'll learn how to design a database that can handle millions of users and large amounts of data.

You already know that application servers can scale horizontally. But here's a new problem:

What if we add 100 application servers, but all 100 servers use the same database, and that database becomes overloaded?

That's what we'll solve today.

## 1. Understand the problem first

Imagine you're building a banking application.

```
                  10,000 Users
                       |
                       v
                 Load Balancer
                       |
             +---------+---------+
             |         |         |
             v         v         v
          Server 1  Server 2  Server 3
             |         |         |
             +---------+---------+
                       |
                       v
                +-------------+
                |  Database   |
                |  OVERLOADED |
                +-------------+
```

Even though we have multiple servers, they all depend on one database.

The database may experience:

- Slow queries
- Too many simultaneous connections
- High CPU and memory usage
- Slow reads and writes
- Request timeouts

Adding more application servers won't necessarily solve the database bottleneck.

Let's learn the solutions one by one.

## 2. SQL vs NoSQL — when should you use each?

First, understand the two broad database categories.

[Intro to SQL](https://images.openai.com/static-rsc-4/T69sYDkBiKZNrrJYKQZgy5x7Lj-yKisabslcEXlixJF38iYFjFBTgNVhs8guXVsgVUaECt7PybeCAXohQmSJrfLsAcioyZE4p671R5kSyzPYfDq7hUWYABjfLhBqzBEQpbkJowgBLI5znYAdA_EX1MIpcQSB7EY3bb9zYZdrOCE?purpose=inline)

SQL database

Organized tables with rows, columns and relationships.

Examples: PostgreSQL, MySQL, Cloud SQL

[How to Use MongoDB for Document-Based NoSQL Databases](https://images.openai.com/static-rsc-4/IRm2vmXiHZ8FjjRv41qvtiLlJUH6jSvThOynsaQnht2To4Y38Z0zFJu6_g7JQWMf90zXUTJzOEv0KrsEllZg1Znr7lm0cSaSwGaH-374XtIB1q76dtr-gY3hWz4LsfZuPDmIb1thjx9wuNcUGKKkT9VASGxU7Q8ztIgf5GX9EcI?purpose=inline)

NoSQL database

Flexible data models, including documents, key-value data and wide-column data.

Examples: Firestore, Bigtable, MongoDB

### Example: Banking application

A relational database might contain these tables:

Customers

| customer_id | name  |
| ----------- | ----- |
| 101         | Rahul |
| 102         | Priya |

Accounts

| account_id | customer_id | balance |
| ---------- | ----------- | ------- |
| 501        | 101         | ₹5,000  |
| 502        | 102         | ₹8,000  |

The `customer_id` connects the two tables.

This structured relationship is useful for banking operations.

### Example: Flexible application data

A document database could store customer preferences as documents:

```
{
  "customer_id": 101,
  "language": "English",
  "notifications": true,
  "preferences": {
    "theme": "dark"
  }
}
```

Different customers can have different preference fields.

Interview rule: Don't say NoSQL is always faster than SQL. Choose a database based on the data model, query patterns, consistency requirements, scale and operational needs.

## 3. Solution 1 — Read Replicas

Suppose most of your database requests are reads.

For example:

- View account details
- View transaction history
- View product information
- Read document metadata

Your main database handles both reads and writes.

```
                  Application
                       |
                       v
                 Main Database
                  READ + WRITE
```

Now imagine that reads are overwhelming the database.

We can introduce read replicas.

A read replica is a copy of database data that can serve read queries.

```
                     Application
                     /         \
              WRITE requests   READ requests
                    |              |
                    v              v
              +-----------+   +-------------+
              |   Primary |-->| Read Replica|
              |  Database |   +-------------+
              +-----------+
                    |
                    +----------> Read Replica 2
```

The primary database handles writes, while replicas handle some reads.

For example:

```
Before:

10,000 read requests → Primary DB

After:

2,000 reads → Primary DB
4,000 reads → Replica 1
4,000 reads → Replica 2
```

These numbers are illustrative. Actual distribution depends on your routing and workload.

### Important limitation

A replica may lag behind the primary database.

Suppose the user transfers money and immediately checks the balance. Reading from a lagging replica could return an older balance.

For critical read-after-write operations, the application may need to read from the primary or use a consistency mechanism appropriate to the database.

Interview answer: If reads are the bottleneck, I would consider read replicas, while keeping consistency requirements in mind.

## 4. Solution 2 — Database Sharding

Now suppose the database contains billions of customer records.

Even after adding read replicas, the primary database might struggle with the amount of data and write traffic.

One approach is sharding.

Sharding means splitting data across multiple database servers.

Imagine we have 9 million customers.

Instead of storing all customer records on one database server:

```
             All 9 Million Customers
                       |
                       v
                +-------------+
                | One Database|
                +-------------+
```

We divide the data into three shards:

```
                 9 Million Customers
                         |
             +-----------+-----------+
             |           |           |
             v           v           v
        +---------+ +---------+ +---------+
        | Shard 1 | | Shard 2 | | Shard 3 |
        +---------+ +---------+ +---------+
        | 1–3 M   | | 3–6 M   | | 6–9 M   |
        +---------+ +---------+ +---------+
```

Each shard owns only a portion of the data.

The application or a routing layer decides which shard should receive a query.

### How does it select the shard?

One approach is to use a customer ID.

For example:

```
shard_number = customer_id % 3
```

If `customer_id = 101`:

```
101 % 3 = 2
```

So the routing rule assigns that customer to shard 2 if shards are numbered 0, 1 and 2.

This is only a teaching example. Real systems need careful shard-key design, and changing the number of shards may require data redistribution.

### Why shard?

- Spread data across multiple database servers.
- Distribute read and write workload.
- Increase capacity beyond one database machine.

### What makes sharding difficult?

- Queries spanning many shards
- Uneven distribution of data
- Moving data when the system grows
- Maintaining consistency across shards
- More complicated operations and monitoring

Important: Sharding is not the first solution for every slow database. First identify the bottleneck and optimize queries, indexes, caching or replicas where appropriate.

## 5. Solution 3 — Database Partitioning

Partitioning and sharding sound similar, but they are not exactly the same.

Partitioning divides a large dataset into smaller pieces, often within one database system. Depending on the database and architecture, those partitions may reside on the same server or be distributed.

Consider a transactions table containing data for several years.

```
Transactions
|
+--- Partition 2024
|
+--- Partition 2025
|
+--- Partition 2026
```

If you query transactions from 2026, the database may be able to skip older partitions.

This is called partition pruning when the database can eliminate partitions irrelevant to the query.

### Common partitioning strategies

- Range partitioning: By date or numeric range.
- List partitioning: By a defined category, such as region.
- Hash partitioning: Based on a hash of a chosen value.

### Partitioning vs sharding

| Partitioning                                     | Sharding                                        |
| ------------------------------------------------ | ----------------------------------------------- |
| Divides a dataset into partitions                | Distributes data across separate database nodes |
| Can exist within one database instance           | Typically involves multiple database nodes      |
| Helps organize and query large datasets          | Helps distribute workload and capacity          |
| Does not automatically increase machine capacity | Can increase capacity across machines           |

The exact meaning depends on the database technology; some distributed databases combine partitioning and sharding.

## 6. Solution 4 — Database Indexing

Before introducing more database servers, consider a simpler optimization: indexes.

Suppose a table contains 10 million users.

You want to find:

```
SELECT *
FROM customers
WHERE customer_id = 500123;
```

Without a useful index, the database may need to examine many rows.

An index is like the index at the back of a textbook. It helps locate the information you need without scanning every page.

```
Without a useful index:
Check many rows → Find customer

With a suitable index:
Look up customer_id → Find customer
```

Indexes can dramatically improve appropriate queries, but they aren't free.

They use storage and can slow down writes because the index must also be maintained.

Interview rule: When a query is slow, inspect the query plan and indexes before assuming you need more database servers.

## 7. Put the solutions together

Let's design a database architecture for a growing application.

Users / Client

Load Balancer

Application Servers

Multiple instances

Cache

Reduce repeated reads where suitable

Primary DB

Writes and critical reads

Read Replicas

Eligible read queries

For workloads that outgrow a single database's capacity, consider partitioning or sharding based on access patterns and consistency needs.

Notice the order of thinking:

1. Optimize slow queries and add appropriate indexes.
2. Use caching for repeatable reads when appropriate.
3. Add read replicas if read load is the bottleneck.
4. Partition large datasets when it improves query efficiency or data management.
5. Consider sharding or a distributed database when one database node cannot meet capacity requirements.

You do not need all five solutions in every architecture.

## 8. Map this to Google Cloud

For your Google Cloud AI Engineer preparation, know these conceptual mappings.

| Requirement                                                   | Google Cloud option |
| ------------------------------------------------------------- | ------------------- |
| Managed relational database                                   | Cloud SQL           |
| Globally scalable relational database with strong consistency | Spanner             |
| Document-oriented NoSQL application data                      | Firestore           |
| Large-scale wide-column workloads                             | Bigtable            |
| Store PDFs, images and other files                            | Cloud Storage       |
| Cache frequently accessed data                                | Memorystore         |
| Analytics over large datasets                                 | BigQuery            |

These services are not interchangeable. For example, BigQuery is primarily an analytics data warehouse, not a direct replacement for an operational banking database.

For a GenAI application, you might use:

```
User
  |
  v
Cloud Run
  |
  +----> Memorystore (cache)
  |
  +----> Cloud SQL (application metadata)
  |
  +----> Cloud Storage (uploaded documents)
  |
  +----> Vector search system (retrieval)
  |
  +----> Gemini (generation)
```

If Cloud SQL becomes the bottleneck, first measure whether the issue is query performance, connections, reads, writes or data volume. Then choose the appropriate solution.

## 9. Google-style interview practice

Imagine the interviewer asks:

"Your application has 10 million users. Database queries are slow. How would you fix the system?"

A strong answer follows a sequence rather than immediately proposing sharding.

1. Understand the workload. Measure read/write volume, latency, slow queries, connection saturation and database resource utilization.
2. Optimize queries. Inspect query plans, add appropriate indexes and reduce unnecessary work.
3. Reduce repeated reads. Introduce a cache if the access pattern and freshness requirements support it.
4. Scale reads. Consider read replicas if reads overload the primary database.
5. Handle growing data. Consider partitioning for query efficiency or sharding/distributed storage when a single node's capacity is insufficient.
6. Protect correctness. For banking or financial transactions, define consistency requirements and avoid stale reads where they could cause incorrect decisions.
7. Monitor results. Measure latency, error rates, resource usage and cost after each change.

This demonstrates that you can diagnose the problem instead of just naming technologies.

## 10. Quick knowledge check

1\. Your database has too many read requests, but writes are manageable. What might help?

Add read replicas

Always shard immediately

Store all data in Cloud Storage

Correct. Read replicas can serve eligible reads and reduce pressure on the primary database.

2\. You need to find a customer by ID in a huge table. What should you investigate first?

Add 100 servers

Query plan and indexes

Replace the database with an LLM

Correct. A suitable index and efficient query may resolve the bottleneck without adding infrastructure.

3\. What is database sharding?

Keeping a second copy only for backups

Splitting data across multiple database nodes

Compressing every row

Correct. Sharding distributes portions of a dataset across database nodes.

4\. Why can reading from a replica be risky immediately after a financial transaction?

The replica may lag behind the primary

Replicas cannot store numbers

The load balancer deletes the data

Correct. Replication lag may cause a read to return stale data.

5\. Which service is designed primarily for analytics at large scale?

BigQuery

Cloud Storage as a SQL engine

Cloud Load Balancing

Correct. BigQuery is Google's data warehouse and analytics platform.

Try again

## What you should remember from Day 7

- Indexing helps locate data efficiently.
- Caching reduces repeated database reads.
- Read replicas distribute eligible read traffic.
- Partitioning divides data into manageable sections.
- Sharding distributes data across multiple database nodes.
- Consistency matters when stale data could lead to incorrect results.

Next: Day 8 — API Design, Rate Limiting and API Gateway. We'll learn how to design APIs, protect them from excessive traffic, and prepare the foundation for production-grade GenAI services.
