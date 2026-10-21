# Day 12 — Networking Fundamentals for System Design

Today, we will learn networking from scratch, using simple examples and Google Cloud architecture. These concepts are important for your Google Cloud AI Engineer system design interviews.

By the end, you'll understand IP addresses, DNS, HTTP/HTTPS, ports, firewalls, VPCs, subnets, and how a GenAI application communicates with a database and Gemini.

## 1. What is networking?

Networking means connecting computers and services so they can communicate and exchange data.

Imagine you build a GenAI chatbot:

User's laptop

Asks a question

Backend API — Cloud Run

Processes the request

Database + Gemini

Retrieve data and generate an answer

These components may run on different machines or in different environments. Networking allows them to communicate.

Let's learn the building blocks.

## 2. IP address — Where is the computer?

An IP address identifies a network interface so devices can send data to the correct destination.

Think of it like a mailing address.

Example IPv4 address:

```
192.168.1.10
```

There are two common categories you'll hear about:

- Private IP: Used for communication within private networks. For example, a database can have a private IP that is not directly reachable from the public internet.
- Public IP: An address that can be routed on the public internet, subject to network configuration and security controls.

Example:

```
Internet
   |
   v
Public-facing application
   |
   v
Private network
   |
   v
Database with private IP
```

Why is this useful? You generally don't want an enterprise database directly exposed to the public internet.

## 3. DNS — How does a name become an address?

DNS stands for Domain Name System.

People prefer names such as:

```
www.example.com
```

Computers use IP addresses to route network traffic. DNS helps resolve a domain name to the appropriate address or service endpoint.

Imagine a user opens your AI application:

```
User enters:
ai.example.com
        |
        v
       DNS
        |
        v
Finds the configured destination
        |
        v
Application endpoint
```

Without DNS, users would often need to remember network addresses instead of convenient names.

Interview question: What happens when a user enters a website's domain name?

A simplified answer: the browser resolves the domain through DNS, establishes a connection to the destination, and sends an HTTP request.

DNS resolution and connection establishment involve additional details, but this is the correct starting mental model.

## 4. HTTP vs. HTTPS

HTTP is a protocol used for communication between clients and web servers.

For example:

```
GET /products
```

This means the client is requesting the `/products` resource.

HTTPS is HTTP protected by TLS, which helps protect data in transit and authenticates the server through its certificate.

| HTTP                                                                         | HTTPS                                       |
| ---------------------------------------------------------------------------- | ------------------------------------------- |
| No TLS protection by itself                                                  | Uses TLS                                    |
| Traffic is not protected by HTTP itself                                      | Protects data in transit                    |
| Not suitable for sensitive internet traffic without another protection layer | Standard choice for modern web applications |

For a banking or enterprise AI application, use HTTPS for client-to-application communication.

Remember: HTTPS protects the connection, but it does not automatically authorize a user to access every API endpoint.

## 5. Ports — Which service should receive the request?

An IP address helps identify the destination network interface. A port helps identify the service or application endpoint on that host.

Think of an office building:

- IP address = building address.
- Port = a particular office or department inside it.

Common ports:

| Port | Common use |
| ---- | ---------- |
| 80   | HTTP       |
| 443  | HTTPS      |
| 22   | SSH        |
| 5432 | PostgreSQL |
| 3306 | MySQL      |

For example, a PostgreSQL database commonly listens on port `5432`.

A firewall or other network control can restrict which sources are allowed to connect to a port.

Important: Knowing a port does not mean the service is publicly accessible. Network routes, firewall rules, service configuration, and authentication also matter.

## 6. Firewall — Who can connect?

A firewall applies rules to network traffic, allowing or blocking connections according to the rules and traffic direction.

Example:

```
Internet
   |
   v
Firewall / network controls
   |
   +---- HTTPS to approved application: ALLOW
   |
   +---- Direct public access to database: BLOCK
```

For a secure GenAI application, you might allow users to reach the web application over HTTPS while restricting database access to approved workloads.

A firewall is not a replacement for application-level authentication or authorization. You often need both.

## 7. VPC — Your private network in Google Cloud

VPC stands for Virtual Private Cloud.

In Google Cloud, a VPC provides a logically isolated network where cloud resources can communicate under configured network rules.

Imagine you are deploying a RAG application:

Users

Google Cloud VPC

Application tier

Backend service

Data tier

Private database connection

A VPC helps organize and control network connectivity. It does not automatically make every resource private or secure; configuration matters.

### What is a subnet?

A subnet is a range of IP addresses within a VPC network.

For example:

```
VPC Network
|
+-- Subnet A: 10.0.1.0/24
|
+-- Subnet B: 10.0.2.0/24
```

Each subnet has its own IP range. In Google Cloud, VPC subnets are regional resources.

You don't need to master CIDR calculations immediately. For now, remember:

- VPC = cloud network.
- Subnet = an IP address range inside that network.

## 8. How does Cloud Run connect to a database?

Let's design a simple enterprise GenAI application using Google Cloud.

```
Employee
   |
   | HTTPS
   v
Cloud Run API
   |
   | Authorized network connection
   v
Cloud SQL / PostgreSQL
   |
   v
Retrieve application data

Cloud Run API
   |
   | Authenticated service request
   v
Gemini on Vertex AI
```

In an actual deployment, you configure the relevant service identity, network connectivity, database access, and model permissions.

A few important principles:

1. Expose only the endpoints that need to be reachable.
2. Prefer private database connectivity where appropriate.
3. Use IAM and database authentication controls correctly.
4. Store credentials in a secure secrets manager if credentials are required.
5. Configure outbound connectivity to external services according to your architecture.

Cloud Run can connect to resources in a VPC using supported networking configurations. The exact setup depends on the Cloud Run environment and destination.

## 9. What is a load balancer?

A load balancer distributes incoming traffic across available backend instances or services according to its configuration.

```
             Users
               |
               v
         Load Balancer
          /    |    \
         v     v     v
      App A  App B  App C
```

Why do we need it?

Suppose 10,000 users access your application. Sending all traffic to a single overloaded instance may result in slow responses or failures.

A load balancer helps distribute traffic, while health checks and appropriate backend configuration help avoid unhealthy destinations.

In Google Cloud, Cloud Load Balancing provides managed load-balancing capabilities.

Note that Cloud Run can also manage incoming traffic and scale instances without requiring you to deploy a separate load balancer for every application.

## 10. Common networking interview questions

1\. What does DNS do?

Encrypts database records

Resolves domain names to network destinations

Stores application passwords

Correct!

2\. What is HTTPS primarily used for?

Protecting HTTP traffic in transit with TLS

Automatically granting database access

Scaling application instances

Correct!

3\. What is a VPC?

A Python package

A virtual cloud network

A database query

Correct!

4\. Which port does PostgreSQL commonly use?

443

22

5432

Correct!

5\. Where should you generally place a sensitive database in an enterprise architecture?

Directly exposed to everyone on the internet

Behind controlled network access with appropriate authentication

With all firewall rules disabled

Correct!

Reset quiz

## Day 12 — Quick revision

- IP address: Identifies a network interface.
- DNS: Resolves domain names to network destinations.
- HTTP/HTTPS: Web communication protocols; HTTPS uses TLS.
- Port: Identifies a service endpoint on a host.
- Firewall: Controls network traffic.
- VPC: A virtual cloud network.
- Subnet: An IP address range within a VPC.
- Load balancer: Distributes incoming traffic across configured backends.

### Next: Day 13 — API Gateway, reverse proxy and service-to-service communication

We'll learn how API Gateway differs from a load balancer, how microservices communicate, and how to design secure APIs for a production GenAI application.
