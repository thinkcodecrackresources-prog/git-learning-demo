# Job Application Tracker

> **Practice project:** Use this as a structure. Replace every detail with what you genuinely built and can defend in an interview.

## Tech Stack

**React, Node.js, PostgreSQL**

## 30-Second Interview Introduction

“I worked on a **Job Application Tracker**. Job seekers often lose track of applications, stages, follow-ups, and notes. My main responsibility was to **create a dashboard for applications and reminders with searchable status history.** I used **React, Node.js, PostgreSQL**. The key outcome was that users could see where each application stood and identify upcoming follow-ups from one dashboard.”

## STAR Explanation

### S — Situation: What problem were you solving?
Job seekers often lose track of applications, stages, follow-ups, and notes.

**Interview version:** “Job seekers often lose track of applications, stages, follow-ups, and notes.”

### T — Task: What was your responsibility?
Create a dashboard for applications and reminders with searchable status history.

**Interview version:** “My responsibility was to create a dashboard for applications and reminders with searchable status history.”

### A — Action: What did you actually build?
Designed CRUD APIs, application-stage model, filters, reminder dates, notes, and dashboard aggregation queries.

**Interview version:** “Designed CRUD APIs, application-stage model, filters, reminder dates, notes, and dashboard aggregation queries.”

### R — Result: What was the outcome?
Users could see where each application stood and identify upcoming follow-ups from one dashboard.

**Interview version:** “Users could see where each application stood and identify upcoming follow-ups from one dashboard.”

> **Important:** Do not invent numbers. If you have real metrics—latency, throughput, users, error reduction, time saved—add them here.

## Architecture

```text
React → Node API → Service → PostgreSQL
```

When explaining architecture, go left to right: **request → validation/auth → business logic → data/storage → async/downstream response**.

## End-to-End Flow

```text
POST /applications → validate company/role → save stage → schedule follow-up metadata → return card
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

**Question:** How would you avoid duplicate applications?

**Strong answer direction:** Use a normalized company/role/job-link key plus a uniqueness check, while still allowing intentional reapplications.

A good challenge answer should cover: **problem → options considered → decision → implementation → result/learning**.

## Common Follow-Up Questions

- How do reminders work?
- How would search work?
- How do you model stages?
- How would you import CSV data?
- How would you add analytics?

## 60-Second Answer Template

“Job seekers often lose track of applications, stages, follow-ups, and notes. My responsibility was to create a dashboard for applications and reminders with searchable status history. Technically, designed CRUD APIs, application-stage model, filters, reminder dates, notes, and dashboard aggregation queries. One important challenge was: how would you avoid duplicate applications? I handled it by use a normalized company/role/job-link key plus a uniqueness check, while still allowing intentional reapplications. Overall, users could see where each application stood and identify upcoming follow-ups from one dashboard.”

## Before You Claim This Project

Make sure you can open the code and explain the schema, one API end-to-end, one failure case, one design trade-off, and one improvement you would make next.
