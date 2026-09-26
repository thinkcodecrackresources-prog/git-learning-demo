# URL Shortener

> **Practice project:** Use this as a structure. Replace every detail with what you genuinely built and can defend in an interview.

## Tech Stack

**Spring Boot, PostgreSQL, Redis**

## 30-Second Interview Introduction

“I worked on a **URL Shortener**. Long URLs are difficult to share and repeated redirects need to be fast. My main responsibility was to **generate unique short codes, persist mappings, and serve low-latency redirects.** I used **Spring Boot, PostgreSQL, Redis**. The key outcome was that users could create compact links and frequently accessed redirects were served with fewer database reads.”

## STAR Explanation

### S — Situation: What problem were you solving?
Long URLs are difficult to share and repeated redirects need to be fast.

**Interview version:** “Long URLs are difficult to share and repeated redirects need to be fast.”

### T — Task: What was your responsibility?
Generate unique short codes, persist mappings, and serve low-latency redirects.

**Interview version:** “My responsibility was to generate unique short codes, persist mappings, and serve low-latency redirects.”

### A — Action: What did you actually build?
Built create/redirect APIs, generated collision-safe codes, added Redis caching, expiry support, and basic click analytics.

**Interview version:** “Built create/redirect APIs, generated collision-safe codes, added Redis caching, expiry support, and basic click analytics.”

### R — Result: What was the outcome?
Users could create compact links and frequently accessed redirects were served with fewer database reads.

**Interview version:** “Users could create compact links and frequently accessed redirects were served with fewer database reads.”

> **Important:** Do not invent numbers. If you have real metrics—latency, throughput, users, error reduction, time saved—add them here.

## Architecture

```text
Client → API → Service → Redis → PostgreSQL
```

When explaining architecture, go left to right: **request → validation/auth → business logic → data/storage → async/downstream response**.

## End-to-End Flow

```text
GET /{code} → check Redis → fallback DB → cache mapping → HTTP redirect
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

**Question:** How do you avoid short-code collisions?

**Strong answer direction:** Use sufficiently large code space and verify uniqueness before persistence; retry generation on collision.

A good challenge answer should cover: **problem → options considered → decision → implementation → result/learning**.

## Common Follow-Up Questions

- How are codes generated?
- Why Redis?
- 301 vs 302?
- How would you handle billions of URLs?
- How do expired links work?

## 60-Second Answer Template

“Long URLs are difficult to share and repeated redirects need to be fast. My responsibility was to generate unique short codes, persist mappings, and serve low-latency redirects. Technically, built create/redirect APIs, generated collision-safe codes, added Redis caching, expiry support, and basic click analytics. One important challenge was: how do you avoid short-code collisions? I handled it by use sufficiently large code space and verify uniqueness before persistence; retry generation on collision. Overall, users could create compact links and frequently accessed redirects were served with fewer database reads.”

## Before You Claim This Project

Make sure you can open the code and explain the schema, one API end-to-end, one failure case, one design trade-off, and one improvement you would make next.
