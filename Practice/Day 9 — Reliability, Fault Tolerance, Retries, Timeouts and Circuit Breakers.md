# Day 9 — Reliability, Fault Tolerance, Retries, Timeouts and Circuit Breakers

Today, we will learn how to design systems that continue working even when something goes wrong.

This is a critical topic for your Google Cloud AI Engineer system design interview, especially when your application depends on Gemini, databases, vector search, and external APIs.

The main question is:

> What happens when one component of your system becomes slow or completely fails?

## 1. First, understand a real problem

Imagine you built a GenAI chatbot.

```
User
  |
  v
Cloud Run API
  |
  v
Gemini API
  |
  v
Response
```

Everything works perfectly during testing.

Suddenly, Gemini takes 30 seconds to respond instead of 2 seconds.

What happens?

- Users wait longer.
- API requests remain active.
- More requests accumulate.
- Resources get consumed.
- Eventually, your application may become overloaded.

Now imagine Gemini becomes completely unavailable.

Your entire chatbot might stop working unless you design for failure.

Let's learn how to prevent that.

## 2. What is reliability?

Reliability means the system continues performing its intended function correctly over time.

For example, a banking application should reliably retrieve balances and process transactions correctly.

Reliability isn't simply about keeping the server running. The system must also return correct results and handle failures safely.

Consider this architecture:

```
User
  |
  v
Application
  |
  v
Database
```

If the database fails, the application might fail too.

The application has a dependency on the database.

A reliable design considers:

- What if the database is unavailable?
- What if Gemini responds slowly?
- What if a request fails halfway through?
- What if a network connection breaks?
- What if the same operation is executed twice?

These questions are the foundation of today's lesson.

## 3. Availability vs reliability

These concepts are related, but different.

| Availability                          | Reliability                                         |
| ------------------------------------- | --------------------------------------------------- |
| Is the system accessible when needed? | Does the system operate correctly and consistently? |
| Focuses on uptime and accessibility   | Includes correct operation and handling failures    |
| Example: API responds to requests     | Example: API returns the correct account balance    |

A service can be available but unreliable. For example, a banking API that responds quickly but returns incorrect balances is available, but not reliable.

## 4. What is fault tolerance?

Fault tolerance means designing a system so that a component failure doesn't necessarily cause the entire system to fail.

Imagine three application instances:

Load Balancer

Server 1

Healthy

Server 2

Failed

Server 3

Healthy

The load balancer routes traffic to healthy instances, assuming sufficient capacity remains.

If Server 2 fails, the other instances can continue handling requests.

But remember: three servers don't guarantee fault tolerance by themselves. You also need enough remaining capacity, appropriate health checks, and protection against shared dependencies failing.

Key idea: A reliable system expects failures and plans how to recover from them.

## 5. What is a timeout?

Suppose your application calls Gemini.

```
Cloud Run API
     |
     | Request
     v
Gemini API
     |
     | No response yet...
     |
     | 30 seconds
     v
Still waiting
```

Should your API wait forever?

No.

A timeout defines how long your application is willing to wait for an operation.

For example, imagine we configure a 5-second timeout for a particular operation.

```
Request sent
     |
     v
Wait up to 5 seconds
     |
     +---- Response arrives → Continue
     |
     +---- No response ------> Timeout
```

The actual timeout should depend on the operation. A simple database lookup might need milliseconds, while an AI generation request may need several seconds or longer.

### Why are timeouts important?

Without timeouts:

- Requests may remain stuck.
- Connections and other resources remain occupied.
- Requests pile up during a downstream outage.
- The application may become overloaded.

With appropriate timeouts, a slow dependency can fail faster from the caller's perspective, allowing the application to respond or recover.

### GenAI example

```
User
  |
  v
Cloud Run
  |
  v
Gemini
  |
  X Response takes too long
  |
  v
Timeout handling
  |
  v
Useful error or fallback
```

A good application might return:

"Your request is taking longer than expected. Please retry shortly."

For long-running work, it may be better to submit a background job and let the user check its status rather than hold a synchronous request open.

## 6. What is a retry?

Suppose your application calls an external API, but a temporary network issue causes the request to fail.

Should the application immediately give up?

Not always.

A retry means attempting the operation again after a failure.

```
Attempt 1
    |
    v
  Failed
    |
    v
Attempt 2
    |
    v
  Failed
    |
    v
Attempt 3
    |
    v
  Success
```

Retries can help with temporary failures, such as intermittent network errors or certain service responses.

However, retries are not a solution for every failure.

For example:

- Invalid credentials generally require fixing authentication.
- Invalid input requires correcting the request.
- A permanent permission denial won't usually be fixed by retrying.
- A rate-limit response may require waiting according to the service's instructions.

### The danger of unlimited retries

Imagine 1,000 requests fail and each retries 100 times.

That's potentially 100,000 additional attempts.

Your already struggling service could receive even more traffic.

This is called a retry storm.

Therefore:

- Set a maximum retry count.
- Retry only appropriate failures.
- Use backoff between attempts.
- Respect service retry guidance.
- Ensure requests have deadlines.

## 7. What is exponential backoff?

Imagine your first attempt fails.

Instead of retrying immediately over and over, wait progressively longer.

Example:

| Attempt | Illustrative wait before retry |
| ------- | ------------------------------ |
| Retry 1 | 1 second                       |
| Retry 2 | 2 seconds                      |
| Retry 3 | 4 seconds                      |
| Retry 4 | 8 seconds                      |

This is exponential backoff.

The waiting time increases exponentially rather than staying constant.

A practical implementation often adds jitter, which means a small random variation in the delay. This prevents many clients from retrying at exactly the same time.

```
Request fails
     |
     v
Wait
     |
     v
Retry
     |
     v
Still failing?
     |
     +---- Yes → Wait longer → Retry if permitted
     |
     +---- No → Continue
```

Retries should also fit within the overall request deadline. There's no value in starting another retry if there isn't enough time left for it to finish.

## 8. Important: Retry + Idempotency

Let's revisit an important concept from Day 6.

Suppose your banking API transfers ₹1,000.

```
Client
  |
  v
Transfer ₹1,000
  |
  v
Bank processes transfer
  |
  v
Response gets lost
```

The client doesn't know whether the transfer succeeded.

It retries:

```
Transfer ₹1,000
```

If the application processes the transfer again, ₹2,000 may be transferred.

That is dangerous.

One solution is to use an idempotency key.

For example:

```
POST /transfers
Idempotency-Key: txn-abc123
```

The server records the result associated with that key. If the same logical request is repeated, it can return the original result rather than executing the transfer again.

The implementation must handle concurrent duplicates and persist the key/result appropriately.

Important: A client-generated key alone isn't enough. The backend must enforce idempotency correctly.

This concept is crucial for payments, banking operations, and agent tools that perform actions.

## 9. What is a circuit breaker?

Now we come to a very important system design pattern.

Suppose Gemini is failing continuously.

Your application keeps sending requests:

```
Request 1 → Gemini → Failed
Request 2 → Gemini → Failed
Request 3 → Gemini → Failed
Request 4 → Gemini → Failed
Request 5 → Gemini → Failed
```

Continuously calling an unhealthy dependency wastes resources and may make recovery harder.

A circuit breaker temporarily stops calls to a dependency when failures cross a configured threshold.

Think of an electrical circuit breaker: when there's a problem, it interrupts the circuit.

### The three states

1\. CLOSED

Requests are allowed normally. The system monitors failures.

Failure threshold exceeded

2\. OPEN

Calls to the failing dependency are blocked temporarily. The application returns an error or uses an appropriate fallback.

After a configured waiting period

3\. HALF-OPEN

A limited number of test requests are allowed. If they succeed, the circuit closes. If they fail, it opens again.

### Example

Your application calls a third-party service.

If enough calls fail:

```
Circuit = OPEN
```

For a short period, the application stops sending normal requests to that service.

After the waiting period:

```
Circuit = HALF-OPEN
```

A test call succeeds:

```
Circuit = CLOSED
```

Normal traffic resumes.

The exact failure threshold and waiting period depend on the application.

Remember the difference:

- Timeout: Stop waiting after a defined duration.
- Retry: Attempt an operation again.
- Circuit breaker: Temporarily stop calls to a failing dependency.

These mechanisms solve different problems and often work together.

## 10. What if Gemini becomes unavailable?

Let's design a more resilient GenAI application.

User

Cloud Run API

Timeout + Bounded Retry + Circuit Breaker

Gemini API

If Gemini is unavailable, return a controlled error or use a suitable, tested fallback where appropriate.

A fallback might be:

- A cached answer, if it is safe and sufficiently fresh.
- A different model, if it supports the same task and is authorized for the data.
- A background job for requests that can be processed later.
- A clear error asking the user to retry.

Do not blindly switch models for sensitive enterprise data. The fallback must satisfy the same security, privacy, quality, and policy requirements.

Also, retries against a rate-limited model can worsen the problem. Respect the model provider's quotas and retry instructions.

## 11. What about RAG failures?

Imagine your architecture is:

```
User
  |
  v
Cloud Run
  |
  v
Vector Search
  |
  v
Gemini
  |
  v
Answer
```

Now suppose vector search is unavailable.

Your system may be unable to retrieve the documents needed for a grounded answer.

A dangerous design would be to pretend the retrieval succeeded and let Gemini invent an answer.

A safer design is to:

1. Detect the retrieval failure.
2. Apply bounded retries for transient errors.
3. Return a clear error or use an explicitly approved fallback.
4. Avoid presenting an ungrounded answer as if it were based on retrieved enterprise documents.
5. Record the failure for monitoring and investigation.

For enterprise RAG, correctly handling failure is better than returning a confidently fabricated answer.

## 12. How to answer the interview question

Interviewer asks:

> "Your GenAI application depends on Gemini. Gemini starts timing out. How will you make your system reliable?"

A strong answer:

"I would first measure the failure rate and latency to confirm whether the issue is with Gemini or another component. I would configure appropriate request deadlines and bounded retries with exponential backoff and jitter for transient failures. If failures persist, I would use a circuit breaker to stop overwhelming the dependency. Depending on the use case, I would return a controlled error, defer long-running work, or use a validated fallback. I would also monitor latency, error rates, token usage, and recovery, while ensuring retries don't cause duplicate side effects."

This answer demonstrates that you understand both failure handling and the trade-offs involved.

## 13. Day 9 knowledge check

1\. What does a timeout do?

Retries forever

Stops waiting after a defined duration

Adds more database replicas

Correct. A timeout bounds how long an operation can wait.

2\. What is exponential backoff?

Increasing retry delays after failures

Sending every request twice

Increasing the size of a database

Correct. Backoff spaces out retries, helping reduce pressure on an unhealthy service.

3\. What does a circuit breaker do when OPEN?

Allows unlimited calls

Temporarily blocks calls to the failing dependency

Deletes the database

Correct. The open state prevents normal calls while the dependency may be unhealthy.

4\. Why is idempotency important for payment APIs?

It makes every request faster

It helps prevent duplicate effects when a request is repeated

It guarantees the model is accurate

Correct. The backend can recognize repeated logical operations and avoid executing them twice.

5\. Your RAG retrieval service fails. What should you avoid?

Logging the error

Returning a controlled error

Pretending retrieval succeeded and presenting an ungrounded answer as verified

Correct. The system should not misrepresent an answer as grounded in documents it failed to retrieve.

Try again

## Today's key takeaways

| Concept             | Purpose                                                  |
| ------------------- | -------------------------------------------------------- |
| Reliability         | Operate correctly and consistently                       |
| Fault tolerance     | Continue functioning despite component failures          |
| Timeout             | Limit how long an operation waits                        |
| Retry               | Attempt a transiently failed operation again             |
| Exponential backoff | Increase the delay between retries                       |
| Idempotency         | Prevent repeated requests from causing duplicate effects |
| Circuit breaker     | Temporarily stop calling an unhealthy dependency         |
| Fallback            | Provide an alternative response or processing path       |

Your design principle for today: Don't assume every dependency will always work. Detect failures, limit their impact, recover safely, and never sacrifice correctness just to keep returning a response.

Next: Day 10 — Observability: Logs, Metrics, Traces, Monitoring and Alerting. You'll learn how to identify what's actually failing in a production AI application instead of guessing.
