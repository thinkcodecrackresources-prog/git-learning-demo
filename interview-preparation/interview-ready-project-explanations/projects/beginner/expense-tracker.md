# Expense Tracker API

> **Practice project:** Use this as a structure. Replace every detail with what you genuinely built and can defend in an interview.

## Tech Stack

**Spring Boot, MySQL, JWT**

## 30-Second Interview Introduction

“I worked on a **Expense Tracker API**. Users needed a simple way to record and understand daily spending. My main responsibility was to **backend apis, authentication, categories, monthly summaries, and database design.** I used **Spring Boot, MySQL, JWT**. The key outcome was that users could securely record, categorize, filter, and summarize expenses.”

## STAR Explanation

### S — Situation: What problem were you solving?
Users needed a simple way to record and understand daily spending.

**Interview version:** “Users needed a simple way to record and understand daily spending.”

### T — Task: What was your responsibility?
Backend APIs, authentication, categories, monthly summaries, and database design.

**Interview version:** “My responsibility was to backend APIs, authentication, categories, monthly summaries, and database design.”

### A — Action: What did you actually build?
Designed REST endpoints, relational tables, validation, JWT security, and aggregate queries for monthly/category summaries.

**Interview version:** “Designed REST endpoints, relational tables, validation, JWT security, and aggregate queries for monthly/category summaries.”

### R — Result: What was the outcome?
Users could securely record, categorize, filter, and summarize expenses.

**Interview version:** “Users could securely record, categorize, filter, and summarize expenses.”

> **Important:** Do not invent numbers. If you have real metrics—latency, throughput, users, error reduction, time saved—add them here.

## Architecture

```text
Client → REST API → Auth Filter → Controller → Service → Repository → MySQL
```

When explaining architecture, go left to right: **request → validation/auth → business logic → data/storage → async/downstream response**.

## End-to-End Flow

```text
POST /expenses: validate JWT → validate payload → verify category → persist expense → return response
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

**Question:** How do you prevent users from reading another user’s expenses?

**Strong answer direction:** Scope every query by authenticated user ID and enforce ownership checks in the service/repository layer.

A good challenge answer should cover: **problem → options considered → decision → implementation → result/learning**.

## Common Follow-Up Questions

- Why MySQL?
- Why JWT?
- How would you implement monthly summaries?
- How would you paginate expenses?
- How would you secure user data?

## 60-Second Answer Template

“Users needed a simple way to record and understand daily spending. My responsibility was to backend APIs, authentication, categories, monthly summaries, and database design. Technically, designed REST endpoints, relational tables, validation, JWT security, and aggregate queries for monthly/category summaries. One important challenge was: how do you prevent users from reading another user’s expenses? I handled it by scope every query by authenticated user ID and enforce ownership checks in the service/repository layer. Overall, users could securely record, categorize, filter, and summarize expenses.”

## Before You Claim This Project

Make sure you can open the code and explain the schema, one API end-to-end, one failure case, one design trade-off, and one improvement you would make next.
