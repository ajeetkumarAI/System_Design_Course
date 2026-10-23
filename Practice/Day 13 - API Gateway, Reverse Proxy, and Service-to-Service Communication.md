# Day 13 — API Gateway, Reverse Proxy, and Service-to-Service Communication

Today, we'll learn how multiple services communicate in a production system and how to protect and manage API traffic. These concepts are useful for Google Cloud AI Engineer system design interviews.

## 1. First, understand the problem

Imagine you build an enterprise GenAI chatbot. It has separate services for authentication, document retrieval, Gemini, and conversation history.

```
                 Employee
                     |
                     v
               GenAI Chatbot
                     |
                     v
                Backend API
                     |
          +----------+----------+
          |          |          |
          v          v          v
      Retrieval    Gemini    Database
       Service     Service   Service
```

As the application grows, you need to manage incoming requests, route them to the right services, apply security rules, and handle failures.

Three important concepts help:

- API Gateway: Manages and controls API access.
- Reverse proxy: Receives requests and forwards them to backend services.
- Service-to-service communication: Allows application components to communicate with each other.

## 2. What is an API Gateway?

An API Gateway is an entry point that can manage requests to backend APIs.

Common responsibilities include:

- Routing requests to the appropriate service.
- Validating authentication tokens, depending on configuration.
- Enforcing rate limits or quotas where supported.
- Applying request and response transformations.
- Providing API management and monitoring.

Example:

```
                    Client
                      |
                      v
                 API Gateway
                      |
             +--------+--------+
             |        |        |
             v        v        v
          Users    Chat API  Reports API
          Service  Service    Service
```

The client uses one managed entry point instead of needing to know every backend service address.

### Example API endpoints

| Endpoint         | Purpose                                |
| ---------------- | -------------------------------------- |
| `POST /chat`     | Ask the chatbot a question             |
| `GET /documents` | Retrieve a list of permitted documents |
| `GET /history`   | View conversation history              |
| `POST /feedback` | Submit answer feedback                 |

The gateway routes requests according to the configured API definition and backend integration.

Important: A gateway does not automatically make every backend operation secure. Backend services must still enforce the required authorization and data-access rules.

## 3. What is a reverse proxy?

A reverse proxy receives requests on behalf of backend servers and forwards them to the appropriate destination.

```
Client
  |
  v
Reverse Proxy
  |
  +----> Backend A
  |
  +----> Backend B
  |
  +----> Backend C
```

A reverse proxy can provide routing, TLS termination, and other capabilities depending on the product and configuration.

Examples include NGINX and managed cloud proxies.

### API Gateway vs. reverse proxy

| Feature                    | API Gateway                 | Reverse Proxy                          |
| -------------------------- | --------------------------- | -------------------------------------- |
| Main focus                 | API management              | Forwarding traffic to backend services |
| Request routing            | Yes                         | Yes                                    |
| Authentication integration | Common capability           | Depends on implementation              |
| API quotas and management  | Common capability           | Depends on implementation              |
| Load balancing             | May integrate with backends | Often supported                        |
| Typical use                | Public or partner APIs      | Backend traffic routing                |

Their capabilities overlap. An API gateway is often implemented using proxy technology, so these are not mutually exclusive categories.

## 4. API Gateway vs. Load Balancer

This is a common system design interview question.

Imagine 100,000 users accessing a GenAI application.

A load balancer distributes traffic across configured backend instances. An API gateway manages API-level access and policies.

```
                  Users
                    |
                    v
               API Gateway
          Authentication / quotas
                    |
                    v
               Load Balancer
                /        \
               v          v
          Backend A    Backend B
               \          /
                \        /
                 v      v
              GenAI Services
```

This is a conceptual design; the exact components and order depend on the deployment.

| API Gateway                     | Load Balancer                    |
| ------------------------------- | -------------------------------- |
| API routing and management      | Traffic distribution             |
| API quotas and policies         | Backend health and availability  |
| API authentication integration  | Routing across backends          |
| API versioning may be supported | Scaling traffic across instances |

Interview answer: Use an API gateway when you need API management, and a load balancer when you need to distribute traffic across backends. A production architecture may use both, but don't add components without a requirement.

## 5. What is service-to-service communication?

In a microservices architecture, each service handles a specific responsibility.

For example:

- Authentication service checks user identity.
- Retrieval service searches documents.
- LLM service coordinates model calls.
- History service stores conversations.

These services communicate using APIs, messaging systems, or other appropriate protocols.

### Synchronous communication

The calling service waits for a response.

```
Chat API
   |
   | HTTP request
   v
Retrieval Service
   |
   | Returns documents
   v
Chat API continues processing
```

Use this when the caller needs the result immediately.

Example: A chatbot needs retrieved documents before generating its answer.

### Asynchronous communication

The caller submits work and continues without waiting for the entire task to finish.

```
Chat API
   |
   v
Message Queue
   |
   v
Background Worker
   |
   v
Process document
```

Use this for long-running work such as document ingestion, batch evaluation, and report generation.

On Google Cloud, Pub/Sub is a common choice for asynchronous messaging.

## 6. How do services authenticate each other?

Suppose your Cloud Run chatbot needs to call another private service.

Should it simply trust the request because it came from another service?

No. The receiving service should verify the caller's identity and whether that caller is authorized.

A common Google Cloud approach is to use service identities and IAM-based access controls where supported.

```
Cloud Run Service A
        |
        | Authenticated request
        v
Private Service B
        |
        v
Verify identity and permission
        |
        v
Return authorized response
```

Apply least privilege: Service A should only receive the permissions it needs.

Also use timeouts, bounded retries, and appropriate error handling to prevent one failing service from cascading failures throughout the system.

## 7. Google Cloud architecture example

Let's design a simple production GenAI application.

Employee / Client

API Gateway

API routing and configured policies

Cloud Run — Chat Backend

Business logic and authorization

Retrieval

Vector database

Model call

Gemini / Vertex AI

Supporting services can have their own access controls, timeouts, and monitoring.

In Google Cloud, API Gateway can expose and manage supported APIs, Cloud Run can host backend services, and Vertex AI can provide access to supported AI models. You configure authentication, network access, and service permissions for your particular deployment.

## 8. Scenario-based interview question

Question: You have a GenAI application with 20 backend services. How would you manage API access and communication?

A strong answer:

1. Define clear APIs for each service.
2. Use an API gateway when centralized API management, quotas, or routing policies are needed.
3. Use a load balancer when traffic needs to be distributed across backend instances.
4. Use synchronous calls when an immediate result is required.
5. Use asynchronous messaging for background jobs and long-running tasks.
6. Authenticate service-to-service calls and enforce least privilege.
7. Apply timeouts, bounded retries, and circuit breakers to protect against failures.
8. Add logs, metrics, and distributed tracing to identify problems.
9. Avoid introducing unnecessary network hops and services that add latency or operational complexity.

## 9. Test your understanding

1\. What is the primary purpose of an API Gateway?

Manage and route API requests

Store vector embeddings

Train an LLM

2\. What is the primary role of a load balancer?

Distribute traffic across configured backends

Write prompts

Authenticate every database row automatically

3\. When is asynchronous communication useful?

When the caller must immediately receive the final result

For long-running background processing

Only for browser CSS

4\. How should a private backend verify another service?

Trust every incoming request

Verify identity and authorization

Disable access controls

5\. A chatbot must retrieve documents before generating an answer. Which communication pattern is a natural fit?

Synchronous request-response

Fire-and-forget with no result handling

No communication

Reset quiz

## Day 13 — Revision

- API Gateway: API management and request routing.
- Reverse proxy: receives and forwards requests to backend services.
- Load balancer: distributes traffic across backends.
- Synchronous communication: wait for the result.
- Asynchronous communication: process work independently.
- Service-to-service security: authenticate callers and enforce permissions.

Next: Day 14 — System Design Practice 1: Design a URL Shortener. We'll combine APIs, databases, caching, load balancing, and scaling into one complete interview-style design before moving on to more complex GenAI systems.
