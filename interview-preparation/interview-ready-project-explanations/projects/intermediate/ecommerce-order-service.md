# E-Commerce Order Service

> **Practice project:** Use this as a structure. Replace every detail with what you genuinely built and can defend in an interview.

## Tech Stack

**Java, Spring Boot, MySQL, Kafka, Redis**

## 30-Second Interview Introduction

“I worked on a **E-Commerce Order Service**. An online store needed a reliable order flow across inventory, payment, and notifications. My main responsibility was to **build order creation and status management while avoiding duplicate orders and inconsistent state.** I used **Java, Spring Boot, MySQL, Kafka, Redis**. The key outcome was that orders moved through a clear lifecycle and downstream systems could react asynchronously to order events.”

## STAR Explanation

### S — Situation: What problem were you solving?
An online store needed a reliable order flow across inventory, payment, and notifications.

**Interview version:** “An online store needed a reliable order flow across inventory, payment, and notifications.”

### T — Task: What was your responsibility?
Build order creation and status management while avoiding duplicate orders and inconsistent state.

**Interview version:** “My responsibility was to build order creation and status management while avoiding duplicate orders and inconsistent state.”

### A — Action: What did you actually build?
Created order APIs, idempotency handling, transactional persistence, event publishing, and Redis-backed read caching.

**Interview version:** “Created order APIs, idempotency handling, transactional persistence, event publishing, and Redis-backed read caching.”

### R — Result: What was the outcome?
Orders moved through a clear lifecycle and downstream systems could react asynchronously to order events.

**Interview version:** “Orders moved through a clear lifecycle and downstream systems could react asynchronously to order events.”

> **Important:** Do not invent numbers. If you have real metrics—latency, throughput, users, error reduction, time saved—add them here.

## Architecture

```text
Client → Order API → Order Service → MySQL → Kafka → Inventory/Payment/Notification
```

When explaining architecture, go left to right: **request → validation/auth → business logic → data/storage → async/downstream response**.

## End-to-End Flow

```text
POST /orders → validate cart → create PENDING order → publish event → downstream processing → update status
```

In an interview, explain one important flow deeply instead of listing every feature. Be ready to say what happens on success, validation failure, dependency failure, and retry.

## What You Should Be Ready to Own

- The APIs or modules you personally implemented
- Database/entity design and why you chose it
- Validation, authentication, and error handling
- One technical decision and its trade-off
- One bug/challenge you debugged
- Testing strategy: unit, integration, and API tests
- How you would scale or improve the system

## Technical Challenge

**Question:** What happens if the client retries the same order request?

**Strong answer direction:** Use an idempotency key mapped to the original order so retries return the existing result instead of creating duplicates.

A good challenge answer should cover: **problem → options considered → decision → implementation → result/learning**.

## Common Follow-Up Questions

- Why Kafka?
- How do you handle payment failure?
- What is idempotency?
- How do you avoid overselling?
- Saga vs distributed transaction?

## 60-Second Answer Template

“An online store needed a reliable order flow across inventory, payment, and notifications. My responsibility was to build order creation and status management while avoiding duplicate orders and inconsistent state. Technically, created order APIs, idempotency handling, transactional persistence, event publishing, and Redis-backed read caching. One important challenge was: what happens if the client retries the same order request? I handled it by use an idempotency key mapped to the original order so retries return the existing result instead of creating duplicates. Overall, orders moved through a clear lifecycle and downstream systems could react asynchronously to order events.”

## Before You Claim This Project

Make sure you can open the code and explain the schema, one API end-to-end, one failure case, one design trade-off, and one improvement you would make next.
