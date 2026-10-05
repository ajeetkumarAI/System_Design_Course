# Day 3 — Database Fundamentals for System Design

Today is very important because in almost every system-design interview, the interviewer will eventually ask:

> **"Where will you store the data, and why did you choose that database?"**

Since you're starting from zero, we'll first understand **what a database actually does**, then learn **SQL vs NoSQL**, and finally make our first database decision.

---

# 1. Start With Our Day 2 Architecture

Yesterday we built:

```text
                    USER
                      |
                      ↓
                    CLIENT
                      |
                      ↓
                LOAD BALANCER
                      |
             ┌────────┼────────┐
             ↓        ↓        ↓
          SERVER    SERVER    SERVER
             └────────┼────────┘
                      ↓
                   DATABASE
```

Today we're going to focus only on:

```text
                   SERVER
                     |
                     ↓
                  DATABASE
```

The question is:

> **What exactly is happening inside that database?**

---

# 2. Why Do We Need a Database?

Suppose our registration system has users:

```text
Ajit
Rahul
Neha
Amit
Priya
```

If the application restarts, we don't want to lose them.

We need permanent storage.

That's the basic purpose of a database.

```text
Application
     |
     ↓
Database
     |
     ↓
Persistent Data
```

**Persistent** means:

> The data remains available even after the application/server restarts.

---

# 3. Think About a Database Like an Excel Sheet

For now, imagine a database as an Excel workbook.

We have a table:

```text
Users

ID | Name  | Email
-----------------------------
1  | Ajit  | ajit@gmail.com
2  | Rahul | rahul@gmail.com
3  | Neha  | neha@gmail.com
```

Each:

* **Row** = one user
* **Column** = one property/attribute
* **Table** = collection of related data

So:

```text
Table
 ↓
Rows + Columns
```

This is the basic idea behind a relational/SQL database.

---

# 4. What Happens When Ajit Registers?

The client sends:

```text
POST /users
```

with:

```text
Name  = Ajit
Email = ajit@gmail.com
```

The backend receives it.

Then the backend tells the database:

> Store this user.

Conceptually:

```text
Backend
   |
   | INSERT USER
   ↓
Database
```

The database stores:

```text
ID | Name | Email
--------------------------
1  | Ajit | ajit@gmail.com
```

---

# 5. What Happens When Ajit Logs In?

Suppose Ajit enters:

```text
Email: ajit@gmail.com
Password: ********
```

The backend needs to find Ajit's information.

So:

```text
Client
  ↓
Login API
  ↓
Backend
  ↓
Database
  ↓
Find Ajit's record
```

The database returns the required information.

Then:

```text
Database
  ↓
Backend
  ↓
Client
```

---

# 6. CRUD

This is a very important term.

CRUD means four basic database operations:

```text
C → Create
R → Read
U → Update
D → Delete
```

### Create

Add a user:

```text
Ajit
```

### Read

Get Ajit's information.

### Update

Change:

```text
Ajit → Ajit Kumar
```

### Delete

Remove the user.

So:

```text
CREATE
READ
UPDATE
DELETE
```

Remember **CRUD**.

You will hear this constantly.

---

# 7. SQL Database

Now we come to the first major database type.

### SQL = Structured Query Language

SQL databases are generally **relational databases**.

Examples:

* PostgreSQL
* MySQL
* Oracle
* SQL Server

Imagine:

```text
Users table

ID | Name | Email
-----------------------
1  | Ajit | ajit@gmail.com
2  | Rahul| rahul@gmail.com
```

And another table:

```text
Orders table

Order_ID | User_ID | Product
------------------------------
101      | 1       | Laptop
102      | 2       | Phone
```

Notice something?

The `User_ID` connects the two tables.

That's where the word **relational** comes from.

---

# 8. Why Do We Use Multiple Tables?

Suppose Ajit places 100 orders.

We don't want to store his complete user information repeatedly:

```text
Order 1 → Ajit, ajit@gmail.com
Order 2 → Ajit, ajit@gmail.com
Order 3 → Ajit, ajit@gmail.com
...
```

Instead:

```text
Users
-----
ID | Name | Email
1  | Ajit | ajit@gmail.com
```

and:

```text
Orders
------
Order_ID | User_ID | Product
101      | 1       | Laptop
102      | 1       | Phone
103      | 1       | Mouse
```

The relationship is:

```text
Users
  |
  | User_ID
  ↓
Orders
```

This is one of the core ideas of relational databases.

---

# 9. What Is a SQL Query?

Suppose we want to find Ajit.

We can conceptually write:

```sql
SELECT * FROM users
WHERE name = 'Ajit';
```

Don't worry about learning SQL today.

Just understand:

> SQL allows us to communicate with a relational database and ask it to create, retrieve, update, or delete data.

---

# 10. Now What Is NoSQL?

Now imagine we don't want to store data in tables.

Instead, we might store something like:

```text
{
    "id": 1,
    "name": "Ajit",
    "email": "ajit@gmail.com"
}
```

This is a different style of storing data.

This is broadly called:

> **NoSQL**

Examples include:

* MongoDB
* Firestore
* DynamoDB
* Cassandra

Don't memorize all of them yet.

The important concept is:

```text
SQL
→ structured relational data

NoSQL
→ flexible/non-relational data models
```

---

# 11. SQL vs NoSQL — Very Simple Example

Suppose our data always looks like:

```text
ID
Name
Email
Phone
```

Very structured.

SQL can be a natural choice:

```text
Users
--------------------
ID | Name | Email
```

But suppose different users have different information.

User 1:

```text
{
    name: "Ajit",
    email: "..."
}
```

User 2:

```text
{
    name: "Rahul",
    email: "...",
    address: "...",
    hobbies: [...]
}
```

User 3:

```text
{
    name: "Neha",
    phone: "...",
    preferences: {...}
}
```

A flexible document-oriented model can be useful.

---

# 12. Don't Make This Mistake

Don't think:

> SQL is good and NoSQL is bad.

Or:

> NoSQL is faster, so always use NoSQL.

That's incorrect.

The correct system-design mindset is:

> **Choose the database based on the requirements and access patterns.**

This is extremely important for interviews.

---

# 13. When Would We Choose SQL?

Suppose we're building:

### Banking system

We might have:

```text
Customer
Account
Transaction
Payment
Loan
```

Relationships matter.

For example:

```text
Customer
   |
   ↓
Account
   |
   ↓
Transaction
```

We need strong consistency and reliable transactions.

A relational database can be a strong choice.

For example:

```text
PostgreSQL
```

or another relational database.

---

# 14. When Might We Choose NoSQL?

Suppose we're building a huge application with:

```text
Millions/billions of users
```

and we need highly scalable access to simple key/document-based data.

A NoSQL database may be appropriate.

For example:

```text
User ID → User Profile
```

or:

```text
Product ID → Product Information
```

Again:

> We don't choose NoSQL simply because it is "faster."

We choose it because its data model, scaling characteristics, availability model, and access patterns fit the problem.

---

# 15. Database Choice Is About Access Patterns

This is a concept I want you to remember.

Suppose interviewer asks:

> "We need to store user profiles."

Don't immediately say:

> PostgreSQL.

Ask:

> How are we going to access the data?

For example:

```text
Find user by ID
```

or:

```text
Find all users belonging to a particular company
```

or:

```text
Find all transactions for a customer
```

or:

```text
Search products by multiple attributes
```

Different access patterns can lead to different design choices.

---

# 16. Google Cloud Mapping

Now let's connect today's concepts to Google Cloud.

### Relational database

Google Cloud offers:

```text
Cloud SQL
```

Cloud SQL supports relational databases such as:

```text
PostgreSQL
MySQL
SQL Server
```

Conceptually:

```text
Application
     ↓
Cloud SQL
```

---

### NoSQL

Google Cloud has services such as:

```text
Firestore
Bigtable
```

They solve different problems.

For example, Firestore is commonly used for application data with a document-oriented model.

Bigtable is designed for very large-scale, low-latency workloads with specific access patterns.

Don't worry about deciding between all of these yet.

We'll learn them gradually.

---

# 17. Database Is Not the Same as Storage

This is another common beginner confusion.

Suppose we have an AI application where users upload PDFs.

Should we put the PDF directly into PostgreSQL?

Usually, no.

We might have:

```text
PDF
 ↓
Cloud Storage
```

and metadata:

```text
Document ID
User ID
File name
Upload date
Location
```

in a database:

```text
Cloud SQL / Firestore
```

So:

```text
Large file
   ↓
Object Storage

Metadata
   ↓
Database
```

This distinction becomes **very important for your GenAI system-design interviews**.

---

# 18. Object Storage

Object storage is designed for files/objects such as:

```text
PDF
Image
Video
Audio
CSV
Excel
Model file
```

In Google Cloud:

```text
Cloud Storage
```

Conceptually:

```text
User
 ↓
Upload PDF
 ↓
Cloud Storage
```

Database might store:

```text
Document ID
User ID
Filename
Storage location
Timestamp
```

---

# 19. Now Think About Your Future RAG System

This concept will become extremely useful later.

Suppose we are building:

> Enterprise RAG chatbot.

We have 1 million PDFs.

We don't want:

```text
PDF → SQL database
```

Instead:

```text
                   Documents
                       |
                       ↓
                 Cloud Storage
                       |
                       ↓
                 Processing
                       |
                       ↓
                 Chunking
                       |
                       ↓
                  Embeddings
                       |
                       ↓
                Vector Database
```

And separately:

```text
Document metadata
       ↓
Database
```

We're not going to learn this architecture today.

I'm showing it so you can see **why understanding basic storage is necessary before learning RAG system design**.

---

# 20. Database Scaling

Now let's return to our original system.

We have:

```text
Users
 ↓
Load Balancer
 ↓
Servers
 ↓
Database
```

Suppose we have 10 million users.

Our application servers can scale horizontally:

```text
Server 1
Server 2
Server 3
Server 4
...
```

But now the database becomes a bottleneck.

```text
          Many Servers
               |
               ↓
        ┌─────────────┐
        │  Database   │ ← Bottleneck
        └─────────────┘
```

This is a very common system-design problem.

---

# 21. Read vs Write

Suppose our application receives:

```text
90,000 READ requests
10,000 WRITE requests
```

That's:

```text
90% READ
10% WRITE
```

Maybe we can separate the workload.

For example:

```text
                 Application
                     |
              ┌──────┴──────┐
              ↓             ↓
            READ          WRITE
              ↓             ↓
         Read Replica    Primary DB
```

This is called **read replication** at a high level.

We'll learn this properly later.

---

# 22. What Is a Database Replica?

Imagine:

```text
Primary Database
       |
       ↓
Replica 1
       |
       ↓
Replica 2
```

The replicas can serve certain read workloads.

So instead of:

```text
10 servers
    |
    ↓
One database
```

we can potentially have:

```text
10 servers
    |
    ↓
Database layer
    |
 ┌──┴────┬──────┐
 ↓       ↓      ↓
Primary Replica Replica
```

This can help with scalability and availability depending on the database and architecture.

---

# 23. One Important Concept: Consistency

Imagine Ajit transfers:

```text
₹10,000
```

from Account A to Account B.

We don't want:

```text
Account A → money removed
Account B → money NOT added
```

We need reliable transactional behavior.

This is one reason relational databases are commonly used in systems requiring strong transactional guarantees.

In system design, you'll hear:

> **Consistency**

At a basic level, think:

> **After an operation, the data should obey the required correctness rules.**

We'll go much deeper into consistency later.

---

# 24. Today's Architecture

Now our architecture can be slightly more realistic:

```text
                       USERS
                         |
                         ↓
                       CLIENT
                         |
                         ↓
                  LOAD BALANCER
                         |
              ┌──────────┼──────────┐
              ↓          ↓          ↓
           SERVER      SERVER      SERVER
              └──────────┼──────────┘
                         |
               ┌─────────┴─────────┐
               ↓                   ↓
          SQL Database        Object Storage
          User metadata       PDFs / Images
```

This is already a useful architecture pattern.

---

# 25. How This Connects to Google Cloud

A possible Google Cloud implementation could be:

```text
                       USERS
                         |
                         ↓
                Cloud Load Balancing
                         |
                         ↓
                      Cloud Run
                         |
              ┌──────────┴──────────┐
              ↓                     ↓
          Cloud SQL            Cloud Storage
          User data            Files
```

Again, **don't memorize this diagram**.

Understand the mapping:

```text
Database
   ↓
Cloud SQL

File/Object Storage
   ↓
Cloud Storage
```

Later we'll introduce:

```text
Pub/Sub
Memorystore
BigQuery
Firestore
Bigtable
Vertex AI
Vector Search
GKE
API Gateway
```

one by one.

---

# 26. Your System Design Decision Framework

When an interviewer asks:

> **Which database would you choose?**

Don't answer immediately.

Think:

### Question 1

What type of data?

```text
Structured?
Semi-structured?
Documents?
Files?
Vectors?
Time-series?
```

### Question 2

How will we access it?

```text
By ID?
Search?
Range query?
Relationships?
Similarity search?
```

### Question 3

How much data?

```text
GB?
TB?
PB?
```

### Question 4

How many requests?

```text
100/sec?
10,000/sec?
1 million/sec?
```

### Question 5

What consistency do we need?

```text
Strong?
Eventual?
```

### Question 6

What availability/scalability requirements?

```text
99.9%?
99.99%?
Global?
Regional?
```

This is how a **system designer thinks**.

---

# 27. The Three Storage Concepts You Must Know Today

Don't leave Day 3 remembering 20 database names.

Remember these three:

### 1. Relational Database

```text
Tables
Rows
Columns
Relationships
SQL
```

Example:

```text
PostgreSQL
```

Google Cloud:

```text
Cloud SQL
```

---

### 2. NoSQL / Document Database

```text
Documents
Flexible schema
Large-scale application data
```

Example:

```text
Firestore
MongoDB
```

---

### 3. Object Storage

For:

```text
PDF
Image
Video
CSV
Excel
Audio
```

Google Cloud:

```text
Cloud Storage
```

---

# 28. The Most Important Comparison

| Requirement                          | Typical choice                |
| ------------------------------------ | ----------------------------- |
| Structured relational data           | SQL                           |
| Strong transactions                  | SQL                           |
| Flexible document data               | NoSQL                         |
| Massive key-value/document workloads | NoSQL                         |
| PDF/Image/Video                      | Object Storage                |
| Large files                          | Object Storage                |
| User/account/transaction data        | SQL often fits                |
| AI documents                         | Object Storage                |
| Vector embeddings                    | Vector database/search system |

Notice the word **"often."**

There is no universal rule.

System design is about **trade-offs**.

---

# 29. Day 3 Interview Question

Imagine I ask you:

> **Design a user registration system for 10 million users. Where would you store user information?**

A beginner answer:

> "I'll use PostgreSQL because SQL is good."

Better answer:

> "The user profile data is structured and has predictable relationships, so I would consider a relational database such as PostgreSQL. On Google Cloud, Cloud SQL for PostgreSQL could be a suitable starting point. The final choice would depend on expected read/write traffic, availability, scaling requirements, and transaction requirements."

That's the type of thinking we want.

---

# 30. Day 3 — What You Should Know Before Moving On

You should now understand:

```text
Database
   ↓
Persistent data
```

```text
CRUD
   ↓
Create
Read
Update
Delete
```

```text
SQL
   ↓
Tables
Relationships
Structured data
Transactions
```

```text
NoSQL
   ↓
Flexible/non-relational models
Scalability for certain workloads
```

```text
Object Storage
   ↓
Files
PDF
Images
Video
CSV
```

And the basic architecture:

```text
                       USER
                         |
                         ↓
                       CLIENT
                         |
                         ↓
                  LOAD BALANCER
                         |
              ┌──────────┼──────────┐
              ↓          ↓          ↓
           SERVER      SERVER      SERVER
              └──────────┼──────────┘
                         |
                ┌────────┴─────────┐
                ↓                  ↓
             DATABASE         OBJECT STORAGE
```

---

# Day 3 Practice — Don't Skip This

Before Day 4, answer these in your own words:

### Q1

What is a database?

### Q2

What does CRUD mean?

### Q3

What is the difference between SQL and NoSQL?

### Q4

Why would we use object storage instead of putting a PDF directly into a relational database?

### Q5

What is the difference between:

```text
Database
vs
Object Storage
```

### Q6

Suppose you are designing a banking application. Would you consider SQL or NoSQL for transactions, and **why**?

### Q7

Suppose you are designing an application where users upload millions of PDFs. Where would you store the PDFs?

### Q8 — System Design

Explain this architecture:

```text
Users
  ↓
Load Balancer
  ↓
Multiple Servers
  ↓
Database
  ↓
Object Storage
```

### Q9 — Google Cloud

Map these concepts:

```text
Relational Database       → ?
Object/File Storage       → ?
Application/Container     → ?
```

---

## One rule for our preparation

From this point onward, **I will not just teach you components**.

For every component, I'll teach you:

**What is it → Why do we need it → What problem does it solve → What happens without it → How does it scale → What are the alternatives → When would Google use it → How do you explain it in an interview.**

That will gradually take you from:

**"I don't know system design"**

to:

**"Give me a GenAI/Agentic AI architecture problem and I can reason through it."**

**Day 4 will be: Caching — why databases become slow, what a cache does, Redis/Memorystore, cache-aside pattern, TTL, cache invalidation, and where caching fits in a Google Cloud architecture.**
