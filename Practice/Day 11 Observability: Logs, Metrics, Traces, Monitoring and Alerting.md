# Day 11 — Observability: Logs, Metrics, Traces, Monitoring and Alerting

## 1. Why do we need observability?

Imagine you built a GenAI chatbot using Gemini and RAG and deployed it on Google Cloud.

```
Employee
   |
   v
Cloud Run API
   |
   v
Vector Database ----> Retrieve Documents
                           |
                           v
                         Gemini
                           |
                           v
                     Final Response
```

One day, users report three problems:

- The chatbot takes 15 seconds to respond.
- Sometimes it returns an error.
- Sometimes the answers are incorrect.

How would you find the cause?

Would you randomly change the code? No. You need information about what happened inside the application.

Observability helps you understand the internal behavior of a system by examining its outputs, events, and measurements.

There are three important pillars:

1. Logs
2. Metrics
3. Traces

## 2. Logs — What happened?

A log is a record of an event that happened inside your application.

For example, when a user asks a question, your application might generate these logs:

```
10:00:01 - Request received
10:00:02 - User authenticated
10:00:03 - Documents retrieved successfully
10:00:04 - Gemini request started
10:00:14 - Gemini response received
10:00:14 - Response sent to user
```

These logs tell us what happened and when.

A Python example:

```
import logginglogging.basicConfig(level=logging.INFO)logging.info("Request received")logging.info("Documents retrieved successfully")logging.error("Gemini request failed")
```

Common log levels:

| Level    | Meaning                                      |
| -------- | -------------------------------------------- |
| DEBUG    | Detailed information for debugging           |
| INFO     | Normal application events                    |
| WARNING  | Something unexpected that may need attention |
| ERROR    | An operation failed                          |
| CRITICAL | A severe problem affecting the application   |

Important: Production logs should not expose passwords, API keys, confidential documents, or sensitive user prompts. Log safe metadata and redact sensitive information.

## 3. Metrics — How is the system performing?

A metric is a numerical measurement collected over time.

Imagine 1,000 employees are using your chatbot. You want to know:

- How many requests are coming in?
- How many requests are failing?
- How long does the chatbot take to respond?
- How much is Gemini costing?
- Is the server running out of memory?

Metrics help answer these questions.

| Metric             | Example              | What it tells you                   |
| ------------------ | -------------------- | ----------------------------------- |
| Request count      | 10,000 requests/hour | Application traffic                 |
| Error rate         | 2%                   | Percentage of failed requests       |
| Latency            | 3 seconds            | How long requests take              |
| CPU utilization    | 85%                  | Processor usage                     |
| Memory utilization | 90%                  | Memory pressure                     |
| Token usage        | 2 million tokens/day | GenAI consumption                   |
| Cache hit rate     | 70%                  | How often cached results are reused |

### Understanding latency

Latency is the time taken to complete a request.

Suppose five requests take these amounts of time:

```
Request 1: 1 second
Request 2: 2 seconds
Request 3: 2 seconds
Request 4: 3 seconds
Request 5: 12 seconds
```

The average is 4 seconds, but that does not tell the whole story: one user waited 12 seconds.

That's why system designers also examine percentiles:

- p50: 50% of requests finish within this time.
- p95: 95% of requests finish within this time.
- p99: 99% of requests finish within this time.

For example, p95 latency of 5 seconds means 95% of measured requests took 5 seconds or less.

## 4. Traces — Where did the time go?

A trace follows one request as it moves through multiple components.

Suppose a chatbot takes 15 seconds to answer.

A distributed trace might show:

```
User Request
     |
     | 200 ms
     v
Cloud Run API
     |
     | 800 ms
     v
Vector Database
     |
     | 13.5 seconds
     v
Gemini
     |
     v
Final Response
```

Now you know that Gemini took most of the time.

Without tracing, you might waste time optimizing your API or database even though neither is the main bottleneck.

### Logs vs. metrics vs. traces

### Logs

Question: What happened?

Example: Gemini returned a timeout error.

### Metrics

Question: How much or how often?

Example: 8% of requests failed in the last five minutes.

### Traces

Question: Where did the request spend its time?

Example: Gemini took 13.5 seconds out of a 15-second request.

Remember this simple rule:

Logs explain events, metrics show trends, and traces reveal the journey of a request.

## 5. Monitoring and alerting

Monitoring means continuously observing your application using metrics, logs, and other signals.

Alerting means notifying the responsible team when a condition requires attention.

For example:

```
Cloud Run Application
        |
        v
Collect Metrics and Logs
        |
        v
Monitoring Dashboard
        |
        v
Check Alert Conditions
        |
        +---- Normal ----> Continue monitoring
        |
        +---- Problem ---> Send Alert
                              |
                              v
                         Engineer investigates
```

Example alert conditions:

- Error rate remains above 5% for five minutes.
- p95 latency exceeds the agreed service target.
- CPU or memory remains dangerously high.
- Queue depth keeps increasing.
- Model failures or spending exceed defined thresholds.

The thresholds should match the application's requirements. Avoid sending an alert for every minor, short-lived fluctuation; alerts should be actionable.

### What is an SLO?

An SLO, or Service Level Objective, is a target for a service's reliability or performance.

For example:

> 99.9% of eligible requests should succeed over a 30-day period.

This is an illustrative target, not a universal requirement. You define the SLO based on business needs.

## 6. The four Golden Signals

A useful starting point for production monitoring is the four Golden Signals.

| Signal     | Meaning                             | Example                        |
| ---------- | ----------------------------------- | ------------------------------ |
| Latency    | How long requests take              | p95 response time is 8 seconds |
| Traffic    | How much demand exists              | 500 requests per second        |
| Errors     | How many requests fail              | 4% return HTTP 500             |
| Saturation | How close resources are to capacity | Memory utilization is 95%      |

These help you identify whether a system is slow, overloaded, failing, or approaching a resource limit.

For GenAI applications, also monitor model-specific usage, costs, token counts, retrieval performance, and quality.

## 7. Observability for GenAI and Agentic AI

Traditional application metrics are not enough for AI applications.

Consider a RAG chatbot:

```
User Question
     |
     v
Retrieve Documents
     |
     v
Build Prompt
     |
     v
Call Gemini
     |
     v
Validate Response
     |
     v
Return Answer
```

At each step, measure relevant information.

| Component           | What to monitor                                 |
| ------------------- | ----------------------------------------------- |
| Retrieval           | Latency, number of chunks, empty results        |
| LLM                 | Latency, token usage, errors, retries, cost     |
| Response            | Relevance, groundedness, correctness            |
| Agent tools         | Tool failures, number of steps, task completion |
| Overall application | Traffic, error rate, p95 latency, availability  |

For AI answer quality, logs and latency metrics are insufficient. Use evaluation datasets, automated evaluations, and human feedback to assess whether answers are correct and grounded in the available evidence.

For agents, watch for repeated tool calls, excessive steps, and loops that increase cost without completing the task.

Also ensure that observability does not become a data-leakage risk: avoid recording sensitive prompts or retrieved documents unnecessarily.

## 8. Google Cloud services to remember

[ProtoEd](https://images.openai.com/static-rsc-4/apWtvq9WQP_lzNIg1hENI87Yt_TSbmNN1whX1LB5YETylVMTttGrGX07gH3tVX9We6a3Nrr1YFsAeXm77C3oGx3XBG55TZb2ga6MSYuvRLJuf_yir4oVrHU8JYIoG-jKVWLiKccBZRpeFSWLLIAKG5Mw8hv6i2zQnVY1SKsqu9c?purpose=inline)

Cloud Logging

Collects and lets you search application logs, including logs from supported Google Cloud services.

[Google Cloud Monitoring Reviews 2026: Details, Pricing, & Features | G2](https://images.openai.com/static-rsc-4/t7qQZJp10VQFJmV7XkG1T82DzgyvRoaveNUI6UrIL1LnA-IKQPHur91F83d78-4_HLyr6tatqHwuUxmeodoFbhNEvtDTmMKxCe6zH2CbPuEqPNXuGOl-2vfLBJdvMNBRt2HaN9KhTYBS6I9U1bJTQce4tpXaJ5jDziBxeK05D3Q?purpose=inline)

Cloud Monitoring

Provides metrics, dashboards, and alerting capabilities.

[Beyond the Black Box: The New, Open, and Analytical World of Google Cloud Trace | by Aditya | Medium](https://images.openai.com/static-rsc-4/tTWp6SowBZnMeOWTXoh8LbvrMxKz56rux4wvlRML70RaXrbD2nPvrVkiOv5o-6RQ-1dTghsDvBDNTmAk4wIGHv6SBhdvmn4cP1guWMzDxH0k1AeoOyUG-UzaQmq2vK0bqfFlSG8CR0JnCb-x5j8kcGUuIUMKhMX-qvIuDUo5GMU?purpose=inline)

Cloud Trace

Helps analyze request latency across instrumented services.

[Application exceptions surfaced automatically | Google Cloud Blog](https://images.openai.com/static-rsc-4/Bd9yy3Yo9mR5K4UXMTV85Y1mQPupkPG1hYs4EDgKp4CJSMPp-Q7l1cG_S_mSWZwwGEFLdxjjGlMzUYMDtAKRPHg-zsgjmP9B0a46Sk4pwhSFsaXCdZzib-peO_405vj1MIMAEXg6xkp5JWgjRVJ5el-jSJj4NV-UfnIm29SG1uI?purpose=inline)

Error Reporting

Helps group and inspect application errors to simplify investigation.

For a Cloud Run-based GenAI application, combine the service's available metrics and logs with application instrumentation. Distributed tracing depends on how the application and its libraries are instrumented.

## 9. Interview scenario

Question: Your production GenAI chatbot takes 15 seconds to respond. How would you investigate?

A strong answer:

1. Check Cloud Monitoring for request traffic, error rate, and latency percentiles.
2. Use Cloud Trace to locate the slow component.
3. If retrieval is slow, inspect vector database latency, query performance, and indexing.
4. If Gemini is slow, inspect model latency, prompt and context size, rate limits, retries, and concurrency.
5. Check application logs for timeouts, errors, and unexpected retries.
6. Optimize the actual bottleneck. Consider streaming, bounded retries, caching where appropriate, and reducing unnecessary context.
7. Set alerts against the application's latency and reliability objectives.
8. Validate answer quality and cost after optimization.

Key interview insight: Do not blindly scale Cloud Run instances when the real bottleneck is model latency or a downstream dependency.

## 10. Test your understanding

1\. Which pillar tells you that Gemini returned a timeout error?

Logs

Metrics

Traces

2\. Which pillar helps you find the slowest component in a single request?

Logs only

Metrics only

Traces

3\. What does p95 latency mean?

95% of requests finish within this time

95 requests failed

The average of all requests

4\. Which Google Cloud service provides metrics, dashboards, and alerting?

Cloud Monitoring

Secret Manager

Cloud Storage

5\. Your chatbot is slow because Gemini takes 13 seconds. What should you investigate first?

Randomly increase database size

Model latency, prompt size, retries, and quotas

Rewrite the entire backend

Reset quiz

## Day 10 — Revision notes

- Observability: Understand the internal behavior of a system.
- Logs: What happened?
- Metrics: How much, how often, or how fast?
- Traces: Where did a request spend its time?
- Monitoring: Continuously track system health.
- Alerting: Notify the team when an actionable condition occurs.
- Golden Signals: Latency, traffic, errors, and saturation.
- GenAI monitoring: Add token usage, model latency, cost, retrieval, agent behavior, and answer quality.

Your next lesson is Day 12 — Networking fundamentals for system design, following the Day 11 security lesson you have already received.
