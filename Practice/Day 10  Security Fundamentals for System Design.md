# Day 10 — Security Fundamentals for System Design

Today, we will learn how to design secure applications, especially GenAI applications on Google Cloud.

By the end of this lesson, you should understand authentication, authorization, IAM, least privilege, secrets management, encryption, and how to apply them to an enterprise AI chatbot.

## 1. Why do we need security?

Imagine you build an AI chatbot at a bank.

The architecture looks like this:

```
Employee
   |
   v
AI Chatbot
   |
   v
Backend API
   |
   +----> Gemini
   |
   +----> Internal Banking Documents
   |
   +----> Database
```

Now imagine these problems occur:

- An unauthorized person accesses the chatbot.
- One employee reads another employee's confidential documents.
- A developer accidentally exposes a Gemini API key on GitHub.
- An attacker steals data while it is being transferred.
- The AI agent performs an action that the user is not permitted to perform.

These are security problems. In system design interviews, you must explain how you would prevent them.

## 2. Authentication vs. Authorization

This is one of the most important interview concepts.

### Authentication — Who are you?

Authentication verifies a user's identity.

Example: An employee logs in using their company account and multi-factor authentication (MFA).

Think: checking someone's identity card.

### Authorization — What can you do?

Authorization determines what an authenticated user is allowed to access.

Example: An employee can read general security documents, but only an authorized security architect can access restricted threat reports.

Think: checking whether that person has permission to enter a restricted room.

### Simple example

Suppose two employees log in successfully.

| Employee   | Authentication | Authorization                            |
| ---------- | -------------- | ---------------------------------------- |
| Employee A | Successful     | Can read public documents                |
| Employee B | Successful     | Can read public and restricted documents |

Both users are authenticated, but they have different permissions.

Interview question: Is logging in enough to protect confidential documents?

Answer: No. We must also enforce authorization on every relevant request and document retrieval.

## 3. What is IAM?

IAM stands for Identity and Access Management.

It controls which users, applications, and services can access which resources.

For example, your GenAI application might need to call Gemini and read documents stored in Cloud Storage.

```
Cloud Run Application
        |
        | Uses its service identity
        v
Google Cloud IAM
        |
        +----> Permission to call the required AI service
        |
        +----> Permission to read approved documents
```

In Google Cloud, you commonly use:

- IAM roles: Define a collection of permissions.
- Service accounts: Provide an identity for an application or workload.
- IAM policies: Grant roles to users, groups, or service identities.

Do not give an application broad administrator permissions when it only needs to read documents and invoke a model.

## 4. Principle of Least Privilege

This means giving a user or application only the permissions it needs to perform its job.

Consider this example.

Bad design

The chatbot's service account has project-wide Owner permissions.

If the chatbot is compromised, an attacker may gain access to far more resources than necessary.

Better design

The service account has only the specific permissions required to invoke the model and read approved data.

Interview tip: Always mention least privilege when discussing cloud security.

## 5. What is a Secret?

A secret is sensitive information that must not be exposed.

Examples:

- API keys
- Database passwords
- Access tokens
- Private credentials
- Encryption keys

Consider this Python code:

```
# Bad practiceapi_key = "my-secret-api-key"
```

Why is this dangerous?

If you commit the code to a public GitHub repository, someone could steal the key and use your account or resources.

A better approach is to store secrets in a dedicated secrets manager.

### Google Cloud Secret Manager

```
Cloud Run Application
        |
        v
Secret Manager
        |
        v
Retrieve authorized secret
        |
        v
Call the required service
```

The application should have permission to access only the specific secret it needs.

For production workloads, prefer secure workload identities and supported authentication mechanisms over long-lived credentials embedded in source code.

## 6. Encryption: Data in Transit vs. Data at Rest

Encryption protects data from unauthorized reading.

Encryption in transit

Protects data while it travels between systems.

Example: An employee's browser communicates with the backend using HTTPS/TLS.

Encryption at rest

Protects stored data.

Example: Documents, database records, and backups are encrypted while stored.

Remember:

- HTTPS/TLS helps protect data moving across a network.
- Encryption at rest protects stored data.
- Encryption does not replace authorization. An authenticated application must still verify whether a user can access a particular record.

## 7. Securing a GenAI application

Now let's apply these concepts to a Retrieval-Augmented Generation (RAG) chatbot.

1\. Employee

Signs in using company identity

2\. Backend authentication and authorization

Verify identity and permissions

3\. Secure document retrieval

Retrieve only documents the employee may access

4\. Gemini

Generate a response from approved context

5\. Secure response

Apply output checks and avoid leaking sensitive information

### What if we skip document-level authorization?

Suppose an employee asks:

> Show me the salary details of the executive team.

The chatbot searches a vector database and retrieves confidential salary documents.

Even if the employee successfully logged in, the system has failed if those documents are not authorized for that employee.

Important: Access control must be enforced during retrieval, not left entirely to the LLM's judgment. A prompt saying “Do not reveal confidential information” is not a replacement for authorization checks.

## 8. Additional GenAI security risks

| Risk                            | Meaning                                               | Protection                                                               |
| ------------------------------- | ----------------------------------------------------- | ------------------------------------------------------------------------ |
| Prompt injection                | Malicious instructions try to manipulate the model    | Treat retrieved content as untrusted; enforce controls outside the model |
| Data leakage                    | Sensitive information appears in responses or logs    | Enforce access controls, minimize sensitive data, redact logs            |
| Excessive agent permissions     | An agent can perform actions beyond its purpose       | Restrict tools and permissions; require approval for sensitive actions   |
| Insecure tool calls             | An agent invokes a tool with unsafe inputs            | Validate inputs and authorize each action                                |
| Denial of service or cost abuse | Excessive requests exhaust resources or model budgets | Rate limits, quotas, budgets, and monitoring                             |

For an agentic AI system, remember that the model should not receive unrestricted access to databases, shell commands, or financial operations.

## 9. Google Cloud security mapping

| Requirement                   | Google Cloud service or approach                      |
| ----------------------------- | ----------------------------------------------------- |
| Manage access and permissions | Cloud IAM                                             |
| Store API keys and passwords  | Secret Manager                                        |
| Encrypt data in transit       | HTTPS/TLS                                             |
| Protect stored data           | Service-supported encryption at rest                  |
| Run an application securely   | Cloud Run with a dedicated service account            |
| Control network exposure      | VPC, firewall rules, and appropriate network controls |
| Detect suspicious activity    | Cloud Audit Logs and Cloud Logging                    |

Exact controls depend on the service and configuration. In an interview, explain the security design rather than simply listing product names.

## 10. Interview scenario

Question: Design a secure enterprise GenAI chatbot on Google Cloud. Employees should access internal documents, and confidential documents must not be exposed to unauthorized users.

A good answer follows this sequence:

1. Authenticate employees using the organization's identity provider.
2. Enforce authorization using roles and document-level permissions.
3. Deploy the backend on Cloud Run using a dedicated service account.
4. Store secrets in Secret Manager rather than source code.
5. Retrieve only documents the requesting employee is authorized to access.
6. Use Gemini through an approved, secured integration.
7. Protect network traffic with HTTPS/TLS and stored data with encryption.
8. Validate agent tool calls and require approval for sensitive actions.
9. Log security events while avoiding unnecessary exposure of prompts and confidential data.
10. Monitor suspicious activity, access failures, and unusual usage.

This demonstrates security at the identity, application, data, model, and operational layers.

## 11. Test your understanding

1\. Authentication answers which question?

What can you access?

Who are you?

How fast is the application?

Correct!

Authentication verifies identity.

2\. What is the principle of least privilege?

Give everyone administrator access

Give only the permissions needed

Disable all authentication

Correct!

This limits damage if a user or application is compromised.

3\. Where should production API secrets be stored?

Public GitHub repository

Hardcoded in Python

A dedicated secrets manager

Correct!

Google Cloud Secret Manager is one option.

4\. Where should RAG document permissions be enforced?

Only in the LLM prompt

During retrieval and access checks

Only after showing the answer

Correct!

Unauthorized documents should not be provided as context to the model.

5\. An AI agent can delete production data without approval. What is the main concern?

Excessive tool permissions

Too many dashboard charts

Encryption at rest

Correct!

Restrict tool capabilities and require approval for high-impact actions.

Reset quiz

## Key points to remember

- Authentication = who you are.
- Authorization = what you can access.
- IAM = managing identities and permissions.
- Least privilege = minimum necessary access.
- Secret Manager = secure storage for application secrets.
- Encryption in transit protects moving data; encryption at rest protects stored data.
- For RAG, enforce document permissions before sending context to the LLM.
- For agents, restrict tool permissions and control high-impact actions.

Next — Day 12: Networking fundamentals for system design: IP addresses, DNS, HTTP/HTTPS, VPCs, subnets, firewalls, and how Cloud Run securely communicates with databases and AI services.
