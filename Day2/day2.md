# Lecture Notes: Monolithic Architecture

## 1. Introduction to Monolithic Architecture

A **Monolithic Architecture** is a traditional software development model where an entire application is built as a single, unified unit. 

* All software components—such as the **User Interface (UI)**, **Business Logic**, and **Data Access Layer**—are tightly coupled and packaged into a single codebase or deployment artifact (e.g., `.jar`, `.war`, `.exe`).
* Monolithic applications typically interact with a single, shared relational or non-relational database.

---

## 2. Core Structure of a Monolith

```
┌──────────────────────────────────────────────────────────┐
│                   Monolithic Application                 │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │ User Interface (Presentation Layer)                 │  │
│  └─────────────────────────┬──────────────────────────┘  │
│                            │                             │
│  ┌─────────────────────────▼──────────────────────────┐  │
│  │ Business Logic Layer (Services / Modules)           │  │
│  │  - User Module   - Order Module   - Payment Module │  │
│  └─────────────────────────┬──────────────────────────┘  │
│                            │                             │
│  ┌─────────────────────────▼──────────────────────────┐  │
│  │ Data Access Layer (ORM / DAO)                      │  │
│  └─────────────────────────┬──────────────────────────┘  │
└────────────────────────────┼─────────────────────────────┘
                             │
                             ▼
              ┌─────────────────────────────┐
              │      Central Database       │
              └─────────────────────────────┘
```

---

## 3. Key Advantages

1. **Simplicity in Early Development:**
   * Easy to set up, build, and start prototyping.
   * Single repository makes code navigation straightforward initially.

2. **Simplified Testing & Debugging:**
   * End-to-end testing can be performed easily in a local environment.
   * Cross-component tracking/debugging doesn't require complex distributed logging tools.

3. **Easier Deployment (Initially):**
   * Single file or package to build and deploy to a server.

4. **High Initial Performance:**
   * Function calls within the same process are much faster than network/API calls between microservices.

---

## 4. Key Disadvantages & Challenges

1. **Scalability Limitations:**
   * **Vertical Scaling Only (or all-or-nothing Horizontal Scaling):** You must scale the entire application, even if only one module (e.g., payment service) is experiencing high load.

2. **Tight Coupling & Code Complexity:**
   * As the codebase grows, it becomes harder to maintain and understand.
   * A change in one module can inadvertently break unrelated modules.

3. **Deployment Risks:**
   * Any small code change requires redeploying the **entire application**.
   * A single bug or memory leak in one module can crash the whole system.

4. **Technology Stack Lock-in:**
   * Hard to adopt new frameworks, languages, or libraries without rewriting the entire codebase.

---

## 5. When to Use Monolithic Architecture?

* **Startups & MVPs:** When fast time-to-market is the top priority.
* **Small / Simple Applications:** Domain complexity is low and traffic is manageable.
* **Small Engineering Teams:** Reduced infrastructure overhead compared to distributed microservices.
