# Day 14 — System Design Practice 1: Design a URL Shortener

Today, we'll combine the concepts you've learned so far: APIs, databases, caching, load balancing, scalability, and reliability.

This is a classic system design interview question. We'll start from the basics and build the design step by step.

## 1. What is a URL shortener?

A URL shortener converts a long URL into a short URL.

For example:

Original URL

`https://example.com/products/electronics/mobile/iphone`

Short URL

`https://short.ly/aB91x`

When a user opens the short URL, the application redirects them to the original URL.

Two main operations are required:

1. Create: Generate a short URL from a long URL.
2. Redirect: Find the original URL and redirect the user.

## 2. Understand the requirements

In a system design interview, don't start drawing architecture immediately. First, clarify what the system must do.

### Functional requirements

These describe what the system should do.

- Accept a long URL.
- Generate a unique short code.
- Store the mapping between short code and long URL.
- Redirect users to the original URL.
- Return an error if a code does not exist.

### Non-functional requirements

These describe how well the system should work.

- Fast redirects.
- High availability.
- Support many users.
- Avoid duplicate short codes.
- Store mappings reliably.
- Protect against malicious URLs and abuse.

For our first design, we'll assume that users don't need accounts and that URLs don't expire by default.

## 3. Design the APIs

An API allows the client and backend to communicate.

We'll create two endpoints.

| Method | Endpoint   | Purpose                      |
| ------ | ---------- | ---------------------------- |
| POST   | `/shorten` | Create a short URL           |
| GET    | `/{code}`  | Redirect to the original URL |

### API 1: Create a short URL

Request:

```
{
  "long_url": "https://example.com/products"
}
```

Response:

```
{
  "short_url": "https://short.ly/aB91x"
}
```

### API 2: Redirect

The user opens:

```
GET /aB91x
```

The backend looks up `aB91x` and returns an HTTP redirect, typically `302`, with the original URL in the `Location` header.

The browser then opens the original URL.

## 4. Draw the basic architecture

Start with the simplest working design.

User / Browser

Backend API

Create codes and handle redirects

Database

Short code → Original URL

The database stores a mapping such as:

| Short code | Original URL                   |
| ---------- | ------------------------------ |
| `aB91x`    | `https://example.com/products` |
| `kT72p`    | `https://example.com/about`    |
| `mR44z`    | `https://example.com/contact`  |

When the user requests `/aB91x`, the backend searches for that code and retrieves its original URL.

This works, but what happens when millions of users access the system?

## 5. How do we generate unique short codes?

We need a code-generation strategy that minimizes collisions.

One approach is to generate a unique numeric ID and encode it using Base62.

Base62 uses:

- `0–9` — 10 characters
- `a–z` — 26 characters
- `A–Z` — 26 characters

Total: 62 characters.

For example, an internal ID could be converted into a short code such as `aB91x`.

This is an illustrative mapping, not a specific encoded ID.

Important: Randomly generating short codes can produce collisions. You need a uniqueness constraint in the database and a safe retry strategy, or use an ID-generation approach that guarantees unique IDs within the system's design.

For a production service, also consider whether sequential or predictable IDs create privacy or abuse risks.

## 6. Which database should we use?

A URL shortener mainly needs to map a short code to an original URL.

A relational database such as PostgreSQL or a key-value/NoSQL database can both work.

For a simple design, we could use a table like this:

```
CREATE TABLE urls (
    short_code VARCHAR(20) PRIMARY KEY,
    original_url TEXT NOT NULL,
    created_at TIMESTAMP NOT NULL
);
```

The primary key on `short_code` allows efficient lookups and enforces uniqueness.

At larger scale, a managed NoSQL database such as Google Cloud Bigtable or another suitable key-value store may be considered depending on traffic patterns, consistency requirements, and operational needs.

The best choice depends on the requirements—not simply on which database is newer.

## 7. How do we improve performance with caching?

Most popular short URLs may be accessed repeatedly.

If every redirect requires a database query, the database can become a bottleneck.

We can introduce a cache.

User requests `/aB91x`

Cache lookup

Cache hit

Return URL immediately

Cache miss

Read database and populate cache

This is the cache-aside pattern you learned earlier.

On Google Cloud, Memorystore can provide a managed Redis-compatible cache, subject to the chosen configuration.

Caching popular URLs reduces repeated database reads and can improve redirect latency.

## 8. How do we scale the application?

Suppose traffic grows from 100 requests per second to 10,000 requests per second.

A single backend instance may not be enough.

We can use multiple instances behind a load balancer.

```
                  Users
                    |
                    v
              Load Balancer
               /    |    \
              v     v     v
           API 1  API 2  API 3
               \    |    /
                v   v   v
                  Cache
                    |
                    v
                 Database
```

Key design choices:

- Horizontal scaling: Add more backend instances.
- Stateless backend: Keep persistent URL mappings in the database rather than only in local memory.
- Caching: Reduce database load.
- Database scaling: Use suitable indexes, replicas where appropriate, or partitioning if justified by scale.
- Monitoring: Track latency, traffic, errors, and resource saturation.

Cloud Run can scale stateless application instances based on configured scaling behavior. A separate load balancer is not always required for a basic Cloud Run deployment.

## 9. What can go wrong?

A good system design answer also covers failure cases.

| Problem                             | Possible solution                                                  |
| ----------------------------------- | ------------------------------------------------------------------ |
| Two requests generate the same code | Enforce uniqueness and safely retry                                |
| Cache is unavailable                | Fall back to the database, with appropriate timeouts               |
| Database is slow                    | Optimize lookups, monitor latency, and scale appropriately         |
| Backend instance fails              | Use multiple instances and health-aware routing                    |
| User requests an unknown code       | Return HTTP `404 Not Found`                                        |
| Malicious URL is submitted          | Validate URLs, apply abuse controls, and block unsafe destinations |
| Traffic spike overloads the service | Rate limiting, autoscaling, caching, and capacity controls         |

A URL shortener is also an abuse-prone service. Production systems may need phishing detection, malware checks, domain restrictions, and rate limits.

## 10. Final architecture

Here is a more complete version of the design.

Users

Load Balancing / API Entry

Stateless Backend Instances

Code generation and redirects

Cache

Fast URL lookups

Database

Durable mappings

Monitoring and alerting cover the components.

### How to explain this in an interview

You can say:

"I would build a URL-shortening service with APIs to create short links and redirect users. I would generate unique codes, store the code-to-URL mapping in a database, and use a cache to accelerate popular redirects. As traffic grows, I would scale stateless backend instances horizontally and monitor latency, errors, and database load. I would also handle collisions, unknown codes, malicious URLs, and component failures."

That is a solid first-pass system design answer.

## 11. Check your understanding

1\. What is the main purpose of the database?

Store short-code-to-URL mappings

Encrypt all internet traffic automatically

Replace the backend API

2\. Why do we use a cache?

To reduce repeated database reads

To generate every URL randomly

To remove the need for persistent storage

3\. What should happen when a short code does not exist?

Return HTTP 404

Return an unrelated URL

Restart the entire system

4\. How can the backend scale to handle more traffic?

Add backend instances

Store everything only in local memory

Remove monitoring

5\. How do you prevent duplicate short codes?

Use a uniqueness constraint and safe code-generation strategy

Assume collisions never happen

Disable the database primary key

Reset quiz

Next: Day 15 — System Design Practice 2: Design a Rate Limiter. We'll learn how to protect APIs from excessive traffic, compare fixed-window and token-bucket algorithms, and apply the design to a GenAI application with quotas and token limits.
