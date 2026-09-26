# Real-Time Chat Application

> **Practice project:** Use this as a structure. Replace every detail with what you genuinely built and can defend in an interview.

## Tech Stack

**React, Spring Boot, WebSocket, Redis, PostgreSQL**

## 30-Second Interview Introduction

“I worked on a **Real-Time Chat Application**. Users needed instant one-to-one and group messaging with message history. My main responsibility was to **build real-time delivery, persistence, unread counts, and reconnect behavior.** I used **React, Spring Boot, WebSocket, Redis, PostgreSQL**. The key outcome was that users could exchange messages in real time and recover conversation history after reconnecting.”

## STAR Explanation

### S — Situation: What problem were you solving?
Users needed instant one-to-one and group messaging with message history.

**Interview version:** “Users needed instant one-to-one and group messaging with message history.”

### T — Task: What was your responsibility?
Build real-time delivery, persistence, unread counts, and reconnect behavior.

**Interview version:** “My responsibility was to build real-time delivery, persistence, unread counts, and reconnect behavior.”

### A — Action: What did you actually build?
Used WebSockets for live messages, persisted messages in PostgreSQL, Redis for presence/unread state, and REST for history pagination.

**Interview version:** “Used WebSockets for live messages, persisted messages in PostgreSQL, Redis for presence/unread state, and REST for history pagination.”

### R — Result: What was the outcome?
Users could exchange messages in real time and recover conversation history after reconnecting.

**Interview version:** “Users could exchange messages in real time and recover conversation history after reconnecting.”

> **Important:** Do not invent numbers. If you have real metrics—latency, throughput, users, error reduction, time saved—add them here.

## Architecture

```text
Client ↔ WebSocket Gateway → Chat Service → PostgreSQL / Redis
```

When explaining architecture, go left to right: **request → validation/auth → business logic → data/storage → async/downstream response**.

## End-to-End Flow

```text
Send message → authenticate socket → persist → publish to recipient/channel → update unread count
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

**Question:** What if a user is offline?

**Strong answer direction:** Persist the message first, mark it unread, and deliver history/unread state when the user reconnects.

A good challenge answer should cover: **problem → options considered → decision → implementation → result/learning**.

## Common Follow-Up Questions

- WebSocket vs polling?
- How do you guarantee ordering?
- How do unread counts work?
- How would you scale sockets?
- How do you secure rooms?

## 60-Second Answer Template

“Users needed instant one-to-one and group messaging with message history. My responsibility was to build real-time delivery, persistence, unread counts, and reconnect behavior. Technically, used WebSockets for live messages, persisted messages in PostgreSQL, Redis for presence/unread state, and REST for history pagination. One important challenge was: what if a user is offline? I handled it by persist the message first, mark it unread, and deliver history/unread state when the user reconnects. Overall, users could exchange messages in real time and recover conversation history after reconnecting.”

## Before You Claim This Project

Make sure you can open the code and explain the schema, one API end-to-end, one failure case, one design trade-off, and one improvement you would make next.
