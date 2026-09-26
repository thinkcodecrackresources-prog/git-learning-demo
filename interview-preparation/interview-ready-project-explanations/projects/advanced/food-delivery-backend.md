# Food Delivery Backend

> **Practice project:** Use this as a structure. Replace every detail with what you genuinely built and can defend in an interview.

## Tech Stack

**Spring Boot, PostgreSQL, Redis, Kafka**

## 30-Second Interview Introduction

“I worked on a **Food Delivery Backend**. A delivery platform needs to coordinate restaurant orders, payments, and delivery status changes. My main responsibility was to **implement order lifecycle and expose reliable status updates to customers and partners.** I used **Spring Boot, PostgreSQL, Redis, Kafka**. The key outcome was that the system had a traceable order lifecycle and services could process updates independently.”

## STAR Explanation

### S — Situation: What problem were you solving?
A delivery platform needs to coordinate restaurant orders, payments, and delivery status changes.

**Interview version:** “A delivery platform needs to coordinate restaurant orders, payments, and delivery status changes.”

### T — Task: What was your responsibility?
Implement order lifecycle and expose reliable status updates to customers and partners.

**Interview version:** “My responsibility was to implement order lifecycle and expose reliable status updates to customers and partners.”

### A — Action: What did you actually build?
Created order state machine, validation, transactional writes, Kafka events, Redis cache for restaurant/menu reads, and retry-safe consumers.

**Interview version:** “Created order state machine, validation, transactional writes, Kafka events, Redis cache for restaurant/menu reads, and retry-safe consumers.”

### R — Result: What was the outcome?
The system had a traceable order lifecycle and services could process updates independently.

**Interview version:** “The system had a traceable order lifecycle and services could process updates independently.”

> **Important:** Do not invent numbers. If you have real metrics—latency, throughput, users, error reduction, time saved—add them here.

## Architecture

```text
App → API Gateway → Order Service → PostgreSQL → Kafka → Payment/Delivery/Notification
```

When explaining architecture, go left to right: **request → validation/auth → business logic → data/storage → async/downstream response**.

## End-to-End Flow

```text
Place order → validate restaurant/items → create order → payment event → confirm → assign delivery → status updates
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

**Question:** How do you keep order state consistent across services?

**Strong answer direction:** Use an explicit state machine, idempotent event consumers, correlation IDs, and compensating actions for failed steps.

A good challenge answer should cover: **problem → options considered → decision → implementation → result/learning**.

## Common Follow-Up Questions

- How would you design order states?
- What if payment succeeds but event fails?
- Why Kafka?
- How do retries work?
- How would you track an order live?

## 60-Second Answer Template

“A delivery platform needs to coordinate restaurant orders, payments, and delivery status changes. My responsibility was to implement order lifecycle and expose reliable status updates to customers and partners. Technically, created order state machine, validation, transactional writes, Kafka events, Redis cache for restaurant/menu reads, and retry-safe consumers. One important challenge was: how do you keep order state consistent across services? I handled it by use an explicit state machine, idempotent event consumers, correlation IDs, and compensating actions for failed steps. Overall, the system had a traceable order lifecycle and services could process updates independently.”

## Before You Claim This Project

Make sure you can open the code and explain the schema, one API end-to-end, one failure case, one design trade-off, and one improvement you would make next.
