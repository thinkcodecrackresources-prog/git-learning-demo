# Cloud File Storage Service

> **Practice project:** Use this as a structure. Replace every detail with what you genuinely built and can defend in an interview.

## Tech Stack

**Java, Spring Boot, Object Storage, PostgreSQL**

## 30-Second Interview Introduction

“I worked on a **Cloud File Storage Service**. Users needed to upload and securely access large files without overloading application servers. My main responsibility was to **build upload/download metadata apis, authorization, and scalable file transfer.** I used **Java, Spring Boot, Object Storage, PostgreSQL**. The key outcome was that large files could be transferred directly to storage while the API controlled access and metadata.”

## STAR Explanation

### S — Situation: What problem were you solving?
Users needed to upload and securely access large files without overloading application servers.

**Interview version:** “Users needed to upload and securely access large files without overloading application servers.”

### T — Task: What was your responsibility?
Build upload/download metadata APIs, authorization, and scalable file transfer.

**Interview version:** “My responsibility was to build upload/download metadata APIs, authorization, and scalable file transfer.”

### A — Action: What did you actually build?
Stored metadata in PostgreSQL, files in object storage, generated pre-signed URLs, validated type/size, and added ownership checks.

**Interview version:** “Stored metadata in PostgreSQL, files in object storage, generated pre-signed URLs, validated type/size, and added ownership checks.”

### R — Result: What was the outcome?
Large files could be transferred directly to storage while the API controlled access and metadata.

**Interview version:** “Large files could be transferred directly to storage while the API controlled access and metadata.”

> **Important:** Do not invent numbers. If you have real metrics—latency, throughput, users, error reduction, time saved—add them here.

## Architecture

```text
Client → Metadata API → PostgreSQL; Client ↔ Object Storage via signed URL
```

When explaining architecture, go left to right: **request → validation/auth → business logic → data/storage → async/downstream response**.

## End-to-End Flow

```text
Request upload → authorize → create metadata → generate signed URL → direct upload → confirm status
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

**Question:** Why not store files in the database?

**Strong answer direction:** Object storage is better suited for large binary objects and allows independent scaling, lifecycle rules, and direct transfer.

A good challenge answer should cover: **problem → options considered → decision → implementation → result/learning**.

## Common Follow-Up Questions

- What are pre-signed URLs?
- How do you secure downloads?
- How do you handle failed uploads?
- How would virus scanning work?
- How do you version files?

## 60-Second Answer Template

“Users needed to upload and securely access large files without overloading application servers. My responsibility was to build upload/download metadata APIs, authorization, and scalable file transfer. Technically, stored metadata in PostgreSQL, files in object storage, generated pre-signed URLs, validated type/size, and added ownership checks. One important challenge was: why not store files in the database? I handled it by object storage is better suited for large binary objects and allows independent scaling, lifecycle rules, and direct transfer. Overall, large files could be transferred directly to storage while the API controlled access and metadata.”

## Before You Claim This Project

Make sure you can open the code and explain the schema, one API end-to-end, one failure case, one design trade-off, and one improvement you would make next.
