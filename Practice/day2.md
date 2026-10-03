# Day 2 — Build Your First System Design

Today we will build your **first complete system from scratch**.

The goal is not to learn many technologies. The goal is to understand **how a request travels through a system**.

By the end of Day 2, you should be comfortable explaining:

```text
User
 ↓
Client
 ↓
API
 ↓
Server
 ↓
Database
 ↓
Response
```

And you should understand **what each box is doing and why it exists**.

---

# 1. Today's Problem

Imagine the interviewer asks:

> **Design a User Registration System.**

A user should be able to enter:

```text
Name: Ajit
Email: ajit@gmail.com
Password: abc123
```

and click:

```text
Register
```

The system should:

1. Receive the user's information.
2. Validate the information.
3. Store the user.
4. Return a success response.

That's it.

Don't think about Google Cloud, Kubernetes, microservices, AI, RAG, etc. yet.

We are starting from the absolute foundation.

---

# 2. First: What is the Client?

The **client** is the thing the user interacts with.

For example:

```text
Web Browser
Mobile App
Desktop Application
```

Suppose we have a website:

```text
┌─────────────────────────────┐
│       Registration          │
│                             │
│ Name:     [ Ajit         ]  │
│ Email:    [ ajit@gmail.com] │
│ Password: [ ************ ]  │
│                             │
│          [ Register ]        │
└─────────────────────────────┘
```

This webpage is the **client**.

So:

> **Client = the part of the system that interacts with the user.**

---

# 3. What happens when I click Register?

This is where system design begins.

You click:

```text
Register
```

The client needs to send the information somewhere.

It sends a request to the backend.

```text
Client
   |
   | HTTP Request
   ↓
Backend
```

For example:

```text
POST /users
```

with data:

```text
{
    "name": "Ajit",
    "email": "ajit@gmail.com",
    "password": "abc123"
}
```

Don't worry about JSON syntax right now.

The important concept is:

> **The client sends a request to the backend.**

---

# 4. What is an API?

Now we need a way for the client to communicate with the backend.

That's where an **API** comes in.

Think of an API as a **door** through which the client communicates with the backend.

```text
                 API
                  ↓
Client ────────→ Backend
```

For example:

```text
POST /users
```

means:

> "Backend, please create a new user."

Another example:

```text
GET /users/123
```

means:

> "Backend, give me the user whose ID is 123."

---

# 5. API is NOT the database

This is a very common beginner confusion.

Don't think:

```text
Client
 ↓
Database
```

Normally, we have:

```text
Client
   ↓
API / Backend
   ↓
Database
```

Why?

Because we don't want users directly accessing the database.

The backend controls:

* validation
* authentication
* authorization
* business logic
* database operations

---

# 6. What is the Backend Server?

The backend is where the application's logic runs.

For our registration system:

```text
Client
   ↓
API
   ↓
Backend Server
```

The backend receives:

```text
Name = Ajit
Email = ajit@gmail.com
Password = abc123
```

Then it might perform:

```text
1. Is name present?
2. Is email valid?
3. Is password strong enough?
4. Does this email already exist?
5. If everything is okay → save user.
```

This is called **business logic**.

---

# 7. What is a Database?

The backend needs somewhere to store the user.

That's the database.

```text
Backend
   ↓
Database
```

We could have a table like:

```text
Users

ID | Name | Email
----------------------------
1  | Ajit | ajit@gmail.com
2  | Rahul| rahul@gmail.com
3  | Neha | neha@gmail.com
```

The database's job is:

> **Persist and retrieve data.**

"Persist" simply means:

> Keep the data so it doesn't disappear when the application stops.

---

# 8. Complete Architecture

Now put everything together.

```text
                    USER
                     |
                     ↓
                WEB / MOBILE
                   CLIENT
                     |
                     | HTTP Request
                     ↓
                 API SERVER
                     |
                     | Business Logic
                     ↓
                  DATABASE
                     |
                     | Data
                     ↓
                 API SERVER
                     |
                     | HTTP Response
                     ↓
                WEB / MOBILE
                     |
                     ↓
                    USER
```

This is your **first real system architecture**.

You should be able to draw this without looking.

---

# 9. Let's Walk Through One Request

This is extremely important.

Imagine Ajit enters:

```text
Name: Ajit
Email: ajit@gmail.com
Password: abc123
```

and clicks:

```text
Register
```

### Step 1 — User

The user clicks:

```text
Register
```

↓

### Step 2 — Client

The browser collects:

```text
Name
Email
Password
```

↓

### Step 3 — API

The client sends:

```text
POST /users
```

↓

### Step 4 — Backend

The backend receives the request.

It validates the data.

↓

### Step 5 — Database

Backend sends something conceptually like:

```text
INSERT USER
```

↓

### Step 6 — Database

Database stores:

```text
1 | Ajit | ajit@gmail.com
```

↓

### Step 7 — Backend

Database says:

```text
Success
```

↓

### Step 8 — API Response

Backend sends:

```text
201 Created
```

↓

### Step 9 — Client

The browser displays:

```text
Registration successful!
```

---

# 10. The Complete Request Flow

Remember this:

```text
USER
  ↓
CLIENT
  ↓
API REQUEST
  ↓
BACKEND
  ↓
BUSINESS LOGIC
  ↓
DATABASE
  ↓
BACKEND
  ↓
API RESPONSE
  ↓
CLIENT
  ↓
USER
```

This flow is one of the foundations of system design.

---

# 11. What is HTTP?

You'll hear this word constantly.

HTTP is basically a protocol used for communication between clients and servers.

For example:

```text
Browser
   ↓
HTTP Request
   ↓
Server
   ↓
HTTP Response
   ↓
Browser
```

You don't need to learn HTTP deeply today.

Just understand:

> **HTTP allows the client and server to communicate.**

---

# 12. GET vs POST

You should know these two today.

## GET

Used when we want to **retrieve data**.

Example:

```text
GET /users/123
```

Meaning:

> Give me user 123.

---

## POST

Used when we want to **send/create data**.

Example:

```text
POST /users
```

Meaning:

> Create a new user.

So remember:

```text
GET  → Retrieve
POST → Create / Send data
```

We'll learn PUT, PATCH and DELETE later.

---

# 13. What is an API Endpoint?

Suppose our application has:

```text
POST /users
GET /users/123
DELETE /users/123
```

Each of these is an **endpoint**.

Think of an endpoint as a specific door into the backend.

For example:

```text
             Backend
                |
       ┌────────┼─────────┐
       ↓        ↓         ↓
   /users    /orders   /products
```

Each endpoint performs a particular operation.

---

# 14. Now Let's Add Login

Our system now needs:

```text
Register
Login
```

We might have:

```text
POST /users
```

for registration.

And:

```text
POST /login
```

for login.

Architecture remains:

```text
Client
  ↓
API
  ↓
Backend
  ↓
Database
```

We don't need a completely new architecture just because we added another feature.

This is an important lesson:

> **Architecture is about components and their responsibilities, not individual features.**

---

# 15. Where Does Authentication Fit?

Suppose Ajit logs in.

```text
Email
Password
```

Backend checks:

```text
Does this user exist?
Is the password correct?
```

Then the system can create an authentication token.

Conceptually:

```text
User
 ↓
Login API
 ↓
Backend
 ↓
Database
 ↓
Authentication
 ↓
Token
 ↓
User
```

We'll study authentication and authorization separately later.

For now, just understand:

> **Authentication answers: "Who are you?"**

And:

> **Authorization answers: "What are you allowed to do?"**

Very important distinction.

---

# 16. Now Let's Think Like a System Designer

Suppose the interviewer says:

> "Design a registration system."

You shouldn't immediately draw 20 boxes.

Start with the requirement.

### Functional requirement

```text
User can register.
```

Then:

```text
User can login.
```

Then:

```text
User can view profile.
```

Now we have:

```text
Functional Requirements
        ↓
Register
Login
View Profile
```

Then think about non-functional requirements:

```text
Scalability
Availability
Security
Performance
Reliability
```

---

# 17. First Interview Architecture

If the interviewer asks:

> "Design a basic user registration system."

You can start with:

```text
                Users
                  |
                  ↓
               Client
                  |
                  ↓
              API Server
                  |
                  ↓
               Database
```

Then explain:

> "The client collects user information and sends it through an API. The backend validates the request and applies business logic. If validation succeeds, it stores the user information in the database and returns the result to the client."

That's already a solid beginner answer.

---

# 18. But Now the Interviewer Makes It Harder

Interviewer:

> "We now have 10 million users."

What problem do you immediately think about?

**Scale.**

Our current architecture:

```text
             Users
               |
               ↓
          One Server
               |
               ↓
            Database
```

Could become:

```text
                    Users
                      |
                      ↓
               Load Balancer
                      |
             ┌────────┼────────┐
             ↓        ↓        ↓
          Server    Server    Server
             └────────┼────────┘
                      ↓
                   Database
```

Now we're introducing:

> **Horizontal scaling**

---

# 19. Why Can't the User Directly Call Server 1?

Because we may have:

```text
Server 1
Server 2
Server 3
```

The user shouldn't need to know which server exists.

Instead:

```text
User
 ↓
Load Balancer
 ↓
Available Server
```

The load balancer hides the complexity from the user.

This is an important system-design principle:

> **Separate the client from the internal infrastructure.**

---

# 20. What Happens If Server 2 Dies?

Imagine:

```text
Server 1 → Working
Server 2 → ❌ Down
Server 3 → Working
```

We don't want the entire application to stop.

The load balancer can detect unhealthy instances and route traffic to healthy ones.

Conceptually:

```text
                Load Balancer
                     |
          ┌──────────┼──────────┐
          ↓          ↓          ↓
       Server 1   Server 2   Server 3
          ✓          ❌          ✓
```

Traffic goes to:

```text
Server 1
Server 3
```

This introduces:

> **High availability**

Meaning:

> The system should remain available even when some components fail.

---

# 21. Today's Important Concepts

By the end of Day 2, you should know these:

| Concept            | Simple meaning                       |
| ------------------ | ------------------------------------ |
| Client             | User-facing application              |
| Backend            | Application logic                    |
| API                | Communication interface              |
| Endpoint           | Specific API operation               |
| HTTP               | Communication protocol               |
| GET                | Retrieve data                        |
| POST               | Create/send data                     |
| Database           | Persistent data storage              |
| Business logic     | Rules the application follows        |
| Load balancer      | Distributes traffic                  |
| Horizontal scaling | Add more servers                     |
| High availability  | Keep system running despite failures |

---

# 22. The Architecture You Should Draw From Memory

Close your notes and try to draw this:

```text
                    USERS
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

Then explain:

```text
User
 ↓
Client
```

The user interacts with the application.

```text
Client
 ↓
Load Balancer
```

The request enters our backend infrastructure.

```text
Load Balancer
 ↓
Server
```

A healthy server processes the request.

```text
Server
 ↓
Database
```

The server reads or writes data.

Then:

```text
Database
 ↓
Server
 ↓
Load Balancer
 ↓
Client
 ↓
User
```

The response travels back.

---

# 23. Your Day 2 Exercise

Don't move to Day 3 yet.

Try to answer these **without looking at the explanation**.

### Question 1

What is the difference between:

```text
Client
Backend
Database
```

---

### Question 2

Why don't we normally allow:

```text
Client → Database
```

directly?

Why do we use:

```text
Client → Backend → Database
```

?

---

### Question 3

What is an API?

Explain it in your own words.

---

### Question 4

What's the difference between:

```text
GET
POST
```

?

---

### Question 5

If one server can handle 1,000 requests/second and we receive 5,000 requests/second, what can we do?

---

### Question 6

What does a load balancer do?

---

### Question 7

What happens if one of our three servers goes down?

---

### Question 8 — Most Important

Explain this architecture in your own words:

```text
                    USERS
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

Don't worry about using technical English perfectly.

**Explain it as if you're explaining it to me as a 10th-class student.**

Once you can answer these 8 questions comfortably, **Day 3 will be Database Fundamentals + SQL vs NoSQL + how to choose a database in system design.**
