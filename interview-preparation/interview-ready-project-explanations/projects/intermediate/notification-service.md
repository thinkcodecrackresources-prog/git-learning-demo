# Notification Service

> **Practice project:** Use this as a structure. Replace every detail with what you genuinely built and can defend in an interview.

## Tech Stack

**Spring Boot, Kafka, Redis, PostgreSQL**

## 30-Second Interview Introduction

“I worked on a **Notification Service**. Multiple products needed email, SMS, and push notifications without duplicating delivery logic. My main responsibility was to **build a reusable asynchronous notification service with preferences and retry handling.** I used **Spring Boot, Kafka, Redis, PostgreSQL**. The key outcome was that product teams could trigger notifications through a common contract and track delivery outcomes.”

## STAR Explanation

### S — Situation: What problem were you solving?
Multiple products needed email, SMS, and push notifications without duplicating delivery logic.

**Interview version:** “Multiple products needed email, SMS, and push notifications without duplicating delivery logic.”

### T — Task: What was your responsibility?
Build a reusable asynchronous notification service with preferences and retry handling.

**Interview version:** “My responsibility was to build a reusable asynchronous notification service with preferences and retry handling.”

### A — Action: What did you actually build?
Consumed events from Kafka, resolved templates/preferences, routed to providers, added retry/backoff and dead-letter handling, and stored delivery status.

**Interview version:** “Consumed events from Kafka, resolved templates/preferences, routed to providers, added retry/backoff and dead-letter handling, and stored delivery status.”

### R — Result: What was the outcome?
Product teams could trigger notifications through a common contract and track delivery outcomes.

**Interview version:** “Product teams could trigger notifications through a common contract and track delivery outcomes.”

> **Important:** Do not invent numbers. If you have real metrics—latency, throughput, users, error reduction, time saved—add them here.

## Architecture

```text
Producer Services → Kafka → Notification Service → Provider Adapters → Email/SMS/Push
```

When explaining architecture, go left to right: **request → validation/auth → business logic → data/storage → async/downstream response**.

## End-to-End Flow

```text
Consume event → validate → check preferences → render template → call provider → store status/retry
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

**Question:** How do you avoid sending the same notification twice?

**Strong answer direction:** Use an event/idempotency key and persist processed delivery records before or atomically with dispatch state.

A good challenge answer should cover: **problem → options considered → decision → implementation → result/learning**.

## Common Follow-Up Questions

- Why asynchronous?
- How do retries work?
- What is a DLQ?
- How do preferences work?
- How do you handle provider outage?

## 60-Second Answer Template

“Multiple products needed email, SMS, and push notifications without duplicating delivery logic. My responsibility was to build a reusable asynchronous notification service with preferences and retry handling. Technically, consumed events from Kafka, resolved templates/preferences, routed to providers, added retry/backoff and dead-letter handling, and stored delivery status. One important challenge was: how do you avoid sending the same notification twice? I handled it by use an event/idempotency key and persist processed delivery records before or atomically with dispatch state. Overall, product teams could trigger notifications through a common contract and track delivery outcomes.”

## Before You Claim This Project

Make sure you can open the code and explain the schema, one API end-to-end, one failure case, one design trade-off, and one improvement you would make next.
