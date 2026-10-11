# Day 8 — API Design, Rate Limiting and API Gateway

Today, we will learn how clients communicate with backend systems, how to design APIs properly, and how to protect our applications from too many requests.

These concepts are important for your Google Cloud AI Engineer interviews because every production GenAI application needs a way to receive requests, validate them, control traffic, and return responses.

By the end of today's lesson, you'll be able to design a basic API architecture for a GenAI application.

## 1. First, what is an API?

API stands for Application Programming Interface.

Think of an API as a waiter in a restaurant.

Customer

Requests food

Waiter = API

Takes the request to the kitchen

Kitchen = Backend

Processes the request

The response travels back to the customer.

In a software application:

```
User
  |
  v
Frontend / Client
  |
  | API Request
  v
Backend
  |
  v
Database
  |
  v
API Response
  |
  v
Frontend displays result
```

For example, when your mobile banking application displays your account balance, the frontend can call a backend API to retrieve it.

The frontend shouldn't directly connect to the database using privileged database credentials.

## 2. What is an API endpoint?

An endpoint identifies a particular API operation.

For example, imagine we build a customer management application.

| Method | Endpoint         | Purpose                       |
| ------ | ---------------- | ----------------------------- |
| GET    | `/customers`     | Retrieve customers            |
| GET    | `/customers/101` | Retrieve customer 101         |
| POST   | `/customers`     | Create a customer             |
| PUT    | `/customers/101` | Replace/update customer 101   |
| PATCH  | `/customers/101` | Partially update customer 101 |
| DELETE | `/customers/101` | Delete customer 101           |

These are examples of REST API conventions. The exact behavior depends on how the API is implemented.

### Understand GET and POST

GET generally retrieves information.

```
GET /customers/101
```

Possible response:

```
{
  "customer_id": 101,
  "name": "Rahul"
}
```

POST generally submits data or creates a resource.

```
POST /customers
```

Request body:

```
{
  "name": "Rahul"
}
```

Possible response:

```
{
  "customer_id": 101,
  "name": "Rahul"
}
```

JSON is a common format for exchanging structured data between a client and an API.

Remember: GET is intended for retrieval, while POST commonly creates resources or initiates operations.

## 3. How does an API request work?

Suppose you're building an AI chatbot.

The user asks:

> Summarize this document.

The frontend sends a request to your API.

```
POST /chat
```

Request body:

```
{
  "message": "Summarize this document",
  "document_id": "doc-123"
}
```

Your backend can then:

1. Authenticate the user.
2. Validate the request.
3. Retrieve relevant document content.
4. Call Gemini with the required context.
5. Return the generated response.

Architecture:

Frontend / Client

API Endpoint: `POST /chat`

Authentication + Validation

RAG / Backend Logic

Gemini

Response to Client

The API is the entry point. It doesn't necessarily perform every operation itself; it can delegate work to other services.

## 4. What is an API Gateway?

Imagine you have 20 different APIs:

```
/users
/customers
/payments
/reports
/chat
/documents
/agents
```

Should every client need to understand the address and security rules of every backend service?

That can become difficult to manage.

An API Gateway provides a common entry point for API traffic and can apply shared policies, depending on the product and configuration.

```
                 Mobile App
                      |
                 Web App
                      |
              External Clients
                      |
                      v
               +-------------+
               | API Gateway |
               +-------------+
                 /    |     \
                /     |      \
               v      v       v
           User API Payment API AI API
```

An API Gateway may support capabilities such as:

- Routing requests to the correct service
- Authentication or integration with identity systems
- Rate limiting or quota enforcement
- Request and response transformation
- Logging and monitoring

The exact capabilities vary by gateway product and configuration.

### API Gateway vs Load Balancer

This is a common interview question.

| Load Balancer                                        | API Gateway                                     |
| ---------------------------------------------------- | ----------------------------------------------- |
| Distributes traffic across healthy backend instances | Provides a managed entry point for APIs         |
| Focuses on availability and traffic distribution     | Focuses on API routing, policies and management |
| Can perform health checks                            | Can enforce API-specific controls               |
| Can operate at different network layers              | Typically operates with API-level awareness     |

They are not necessarily alternatives. An architecture can use both.

## 5. What is rate limiting?

Suppose your GenAI API allows users to ask questions.

One user writes a script that sends 10,000 requests in one minute.

```
User A → 10 requests
User B → 20 requests
User C → 10,000 requests
```

What could happen?

- Your application becomes overloaded.
- Gemini usage increases.
- Your bill increases.
- Other users experience slower responses.
- You may hit model quotas or rate limits.

We need to control how many requests a client can make.

This is called rate limiting.

For example:

```
Maximum: 60 requests per minute per user
```

If a user sends 40 requests:

```
40 < 60
Allowed
```

If that user sends another 30 requests within the same applicable window:

```
70 > 60
Some requests may be rejected or delayed
```

The exact behavior depends on the rate-limiting policy.

A typical response when a limit is exceeded is HTTP `429 Too Many Requests`.

### Why rate limiting matters for GenAI

LLM calls can be expensive and subject to provider quotas. Rate limiting helps protect both your service and its budget.

However, a request count alone isn't always enough. One request might consume 100 tokens, while another consumes 50,000 tokens.

For GenAI systems, you may also need limits based on:

- Tokens per minute
- Requests per minute
- Concurrent requests
- Per-user or per-tenant budgets
- Maximum input size
- Maximum output tokens

## 6. What happens when too many requests arrive?

Let's combine the concepts.

```
                    Users
                      |
                      v
                +-----------+
                | API Gateway|
                +-----------+
                      |
               Rate Limiting
                      |
                      v
                Load Balancer
                      |
             +--------+--------+
             |        |        |
             v        v        v
          Server 1 Server 2 Server 3
                      |
                      v
                 Gemini API
```

Imagine the system receives more requests than your application can safely handle.

The API Gateway or another enforcement layer can reject excessive traffic. The load balancer distributes allowed traffic. Application instances handle requests and invoke downstream services.

Important: Rate limiting and scaling solve different problems.

- Scaling increases the system's ability to handle workload.
- Rate limiting controls how much workload is admitted.

A scalable system can still need rate limiting to protect expensive downstream services.

## 7. What is authentication vs authorization?

You learned these terms earlier. Now let's connect them to APIs.

Authentication: Who are you?

Example: The user signs in and provides a valid identity token.

Authorization: What are you allowed to do?

Example: A user may access only documents belonging to their department.

Consider this request:

```
GET /documents/confidential-123
```

Even if the user is authenticated, the backend must check whether the user has permission to access that document.

For enterprise GenAI systems, this is especially important. The retrieval layer must enforce document-level access permissions; otherwise, an AI response could reveal confidential information.

A secure flow might be:

```
Request
   |
   v
Authenticate User
   |
   v
Check Permissions
   |
   v
Retrieve Authorized Documents
   |
   v
Call LLM
   |
   v
Return Response
```

Never rely only on the prompt saying, "Do not reveal confidential documents." Access control must be enforced by the application and data layer.

## 8. Google Cloud mapping

Here is how these concepts map to Google Cloud services.

| Requirement                                   | Example service                                     |
| --------------------------------------------- | --------------------------------------------------- |
| Run your API backend                          | Cloud Run                                           |
| Manage API access, policies and routing       | Apigee or API Gateway, depending on requirements    |
| Distribute incoming traffic                   | Cloud Load Balancing                                |
| Manage user identities and application access | Cloud IAM and appropriate identity services         |
| Store application records                     | Cloud SQL or another suitable database              |
| Call Google's Gemini models                   | Gemini through supported Google Cloud AI interfaces |

You don't need to put every service in every architecture.

For example, a small application might use Cloud Run directly with authentication and application-level controls. A larger enterprise API platform may use Apigee for API management.

## 9. Mini system design: Enterprise GenAI Chatbot

Let's design a chatbot that employees use to ask questions about internal policies.

Requirements:

- Only authenticated employees can use it.
- Each employee can make a limited number of requests.
- The chatbot retrieves documents the employee is authorized to access.
- Gemini generates an answer based on the retrieved content.
- Excessive requests are controlled.

Employee / Client

API Gateway / API Entry

Authentication + Rate Limiting

Cloud Run Backend

Retrieval

Authorized documents

Gemini

Answer generation

Response + Logging + Monitoring

### How would you explain this in an interview?

You could say:

"I would expose the chatbot through a secured API. The API entry layer would enforce appropriate authentication and request quotas. A Cloud Run backend would validate requests, check document-level permissions, retrieve relevant authorized content, and call Gemini to generate the answer. I would monitor latency, error rates, token usage and cost. I would also configure limits and timeouts to protect downstream services."

Notice that you're explaining the responsibilities of each component and why it exists, rather than just listing product names.

## 10. Practice questions

1\. Which HTTP method is generally used to retrieve a resource?

POST

GET

DELETE

Correct. GET is generally used to retrieve a resource.

2\. What is the main purpose of an API Gateway?

Only store database records

Provide a managed entry point for APIs and enforce policies

Generate embeddings automatically

Correct. An API Gateway can route API traffic and apply configured API policies.

3\. What does HTTP 429 usually mean?

Unauthorized

Resource not found

Too many requests

Correct. 429 indicates that the client has sent too many requests under the applicable policy.

4\. Why might a GenAI API need token-based quotas as well as request limits?

All requests use the same number of tokens

Requests can have very different token costs

Tokens replace authentication

Correct. A short request and a very long prompt can have very different costs and resource demands.

5\. A user is authenticated but tries to retrieve another department's confidential document. What must the backend check?

Authorization and document permissions

Only the user's IP address

Only the LLM's prompt

Correct. Authentication identifies the user; authorization determines whether they can access that document.

Reset quiz

## Day 8 — What to remember

- API: The interface through which clients communicate with backend services.
- Endpoint: A specific operation, such as `POST /chat`.
- API Gateway: A managed entry point for API routing and policies.
- Load Balancer: Distributes traffic across backend instances.
- Rate limiting: Controls how much traffic a client can send.
- Authentication: Identifies the user.
- Authorization: Checks what the user can access.
- GenAI-specific control: Manage token usage, concurrency, quotas, timeouts and cost.

Next: Day 9 — Reliability, Fault Tolerance, Retries, Timeouts and Circuit Breakers. We'll learn how to design systems that continue behaving safely when a database, API, or AI model becomes slow or unavailable.
