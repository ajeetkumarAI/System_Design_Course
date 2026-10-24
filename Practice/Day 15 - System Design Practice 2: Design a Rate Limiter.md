# Day 15 — System Design Practice 2: Design a Rate Limiter

Today, we'll learn how to protect an API from excessive requests. This is especially important for Google Cloud AI Engineer interviews because GenAI applications need to control traffic, model usage, and cost.

## 1. What is a rate limiter?

A rate limiter controls how many requests a user or client can make within a specified period.

Imagine your GenAI chatbot has 1,000 users. One user sends 10,000 requests in one minute.

Without protection, that user could overload your backend, consume your Gemini quota, and increase your cloud bill.

A rate limiter can enforce a rule such as:

Maximum 10 requests per user per minute.

User sends requests

Rate Limiter

Check the user's request count

Allowed

Process request

Limited

Return HTTP 429

HTTP `429 Too Many Requests` is the standard response when a client exceeds a rate limit.

## 2. Understand the problem with an example

Suppose the rule is 5 requests per minute.

| Request number | Result   |
| -------------- | -------- |
| 1              | Allowed  |
| 2              | Allowed  |
| 3              | Allowed  |
| 4              | Allowed  |
| 5              | Allowed  |
| 6              | Rejected |
| 7              | Rejected |

When the applicable time window resets, requests can be allowed again according to the algorithm.

The important question is: How do we count requests and decide when to reject them?

Let's learn three common algorithms.

## 3. Algorithm 1 — Fixed Window Counter

The fixed-window algorithm divides time into fixed intervals.

For example, every minute is a separate window.

Limit: 5 requests per minute.

```
Window 1: 10:00:00 – 10:00:59

Request count = 5
Allowed: 5
Rejected: 6th request

Window 2: 10:01:00 – 10:01:59

Counter resets
Allowed: 5 new requests
```

### How does it work?

1. Identify the user.
2. Identify the current time window.
3. Read that user's counter.
4. If the counter is below the limit, increment it and allow the request.
5. Otherwise, reject the request.

### Problem: The boundary burst

Suppose the limit is 5 requests per minute.

A user sends:

- 5 requests at 10:00:59.
- 5 more requests at 10:01:00.

The user sends 10 requests within approximately one second, even though each minute's counter stays within its limit.

This is the main weakness of fixed windows.

Advantages: Simple, fast, and memory-efficient.

Disadvantages: Allows bursts around window boundaries.

## 4. Algorithm 2 — Sliding Window

A sliding-window algorithm considers requests within a moving time interval.

Suppose the limit is 5 requests in the previous 60 seconds.

At 10:01:00, the system checks requests from approximately 10:00:00 to 10:01:00 rather than simply resetting a counter at the minute boundary.

```
Current time: 10:01:00

Look back 60 seconds
        |
        v
Count recent requests
        |
        +---- Count < 5 ---> Allow
        |
        +---- Count >= 5 --> Reject
```

There are different implementations:

- Sliding-window log: Records individual request timestamps.
- Sliding-window counter: Estimates the moving-window count using adjacent fixed windows.

The log approach can be accurate but requires more storage per request. The counter approach is more memory-efficient but may be approximate.

Advantages: Better control over bursts across time boundaries.

Disadvantages: More complex than a basic fixed-window counter.

## 5. Algorithm 3 — Token Bucket

The token-bucket algorithm is widely useful when a system should allow small bursts but control sustained traffic.

Imagine a bucket that holds tokens.

- The bucket can hold at most 5 tokens.
- A new token is added every 12 seconds.
- Every request consumes one token.
- If the bucket is empty, the request is rejected or delayed.

## Token bucket

Capacity: 5 tokens

Request arrives

If a token is available, consume one and allow the request.

Tokens refill over time

The bucket gradually recovers its capacity, allowing future requests.

With 5 available tokens, the user can send 5 requests in a burst. After that, the user must wait for tokens to refill.

Advantages: Supports controlled bursts and limits sustained request rates.

Disadvantages: Requires careful handling of token state and concurrency in distributed systems.

## 6. Compare the three algorithms

| Feature                  | Fixed window                   | Sliding window               | Token bucket                 |
| ------------------------ | ------------------------------ | ---------------------------- | ---------------------------- |
| Easy to implement        | Yes                            | Moderate                     | Moderate                     |
| Controls boundary bursts | Poorly                         | Better                       | Controls bursts by capacity  |
| Allows bursts            | Yes, including boundary bursts | Depends on implementation    | Yes, up to bucket capacity   |
| Memory use               | Low                            | Depends on implementation    | Low per tracked client       |
| Good starting use case   | Simple API quotas              | Strict rolling-window quotas | APIs that allow small bursts |

There is no universally best algorithm. Choose based on traffic patterns, fairness requirements, complexity, and cost.

## 7. Where should we store the counters?

In a single backend instance, we could store counters in memory. But what happens when we have multiple instances?

```
                Users
                  |
                  v
             Load Balancer
              /         \
             v           v
          Server A    Server B
          Counter=4   Counter=4
```

If each server maintains its own counter, a user could make requests to both servers and exceed the intended limit.

For a distributed application, you often need shared rate-limit state or a centralized enforcement mechanism.

One possible architecture is:

```
Users
  |
  v
API Gateway / Backend
  |
  v
Shared Rate-Limit State
(e.g. Redis-compatible store)
  |
  +---- Within limit ---> Process request
  |
  +---- Over limit -----> HTTP 429
```

On Google Cloud, you can consider API Gateway quotas where they fit the requirement, or a custom rate limiter using a suitable shared store such as Memorystore.

The exact implementation must account for atomic counter updates, concurrency, state expiration, and behavior when the limiter itself is unavailable.

## 8. Design a rate limiter for a GenAI application

Now apply the same concept to a Gemini-powered chatbot.

A request consumes more than backend CPU. It may also consume model quota, tokens, and money.

A robust design can enforce several limits:

- Requests per user per minute.
- Requests per organization per minute.
- Maximum input tokens per request.
- Maximum output tokens per request.
- Maximum concurrent model calls.
- Daily or monthly usage budgets.

User

API Gateway / Backend

Authentication and request validation

Rate and quota checks

User, organization, tokens, concurrency

RAG + Gemini

Process allowed requests

### Why do we need token limits?

Consider two users:

- User A submits a short question requiring 100 input tokens.
- User B submits a huge document requiring 100,000 input tokens.

Both have sent one request, but their resource consumption differs significantly.

Therefore, request-count limits alone may not protect your application from excessive model usage.

A practical design uses request limits alongside token budgets, concurrency controls, provider quotas, and monitoring.

## 9. What happens when a request is rejected?

Return an HTTP `429` response with a helpful message. When possible, provide a `Retry-After` header to indicate when the client may try again.

Example:

```
HTTP/1.1 429 Too Many Requests
Retry-After: 30
Content-Type: application/json
```

```
{
  "error": "rate_limit_exceeded",
  "message": "Too many requests. Please try again later."
}
```

Clients should not retry immediately in a tight loop. They should respect retry guidance and use appropriate backoff.

## 10. Interview answer

Question: Design a rate limiter for a high-traffic GenAI API.

A strong answer:

1. Clarify whether the limit is per user, API key, organization, or global.
2. Define the quota, time window, and acceptable burst behavior.
3. Choose an algorithm, such as token bucket for controlled bursts.
4. Store state in a shared system or enforce quotas at a suitable gateway.
5. Ensure concurrent requests cannot bypass the limit through race conditions.
6. Return HTTP 429 when limits are exceeded.
7. Apply separate controls for model tokens, concurrency, and spending.
8. Monitor rejected requests, latency, and usage.
9. Define fail-open or fail-closed behavior if the rate-limiting system becomes unavailable, based on the security and availability requirements.

## 11. Test your understanding

1\. What does HTTP 429 mean?

The request succeeded

Too many requests

The server was permanently deleted

Correct!

2\. Which algorithm can allow small bursts while limiting sustained traffic?

Token bucket

Fixed window only

DNS

Correct!

3\. Why can local counters fail in a multi-instance application?

Each instance may track a separate count

They automatically share state

They eliminate duplicate requests

Correct!

4\. Why do GenAI applications need token limits in addition to request limits?

Every request consumes exactly the same resources

Requests can consume very different numbers of tokens

Tokens replace authentication

Correct!

5\. What should a client generally do after receiving a 429 response?

Retry continuously without waiting

Respect retry guidance and back off

Send more requests immediately

Correct!

Reset quiz

## Day 15 — Quick revision

- Rate limiter: Controls how frequently clients can make requests.
- Fixed window: Simple counters in fixed time intervals.
- Sliding window: Tracks or estimates usage across a moving interval.
- Token bucket: Allows controlled bursts while limiting sustained traffic.
- HTTP 429: Too many requests.
- Distributed rate limiting: Requires consistent enforcement across multiple instances.
- GenAI protection: Combine request quotas with token, concurrency, and cost controls.

Next: Day 16 — Design a Notification System. We'll learn queues, asynchronous processing, retries, delivery status, and how to scale notifications to millions of users.
