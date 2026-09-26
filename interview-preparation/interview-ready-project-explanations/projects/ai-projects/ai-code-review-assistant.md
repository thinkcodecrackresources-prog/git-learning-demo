# AI Code Review Assistant

> **Practice project:** Use this as a structure. Replace every detail with what you genuinely built and can defend in an interview.

## Tech Stack

**Python, FastAPI, Git Diff Parser, LLM/RAG**

## 30-Second Interview Introduction

“I worked on a **AI Code Review Assistant**. Large pull requests are time-consuming to review and repetitive issues can be missed. My main responsibility was to **summarize diffs and flag likely correctness, testing, and maintainability concerns for human review.** I used **Python, FastAPI, Git Diff Parser, LLM/RAG**. The key outcome was that reviewers got a focused first pass while retaining final decision-making responsibility.”

## STAR Explanation

### S — Situation: What problem were you solving?
Large pull requests are time-consuming to review and repetitive issues can be missed.

**Interview version:** “Large pull requests are time-consuming to review and repetitive issues can be missed.”

### T — Task: What was your responsibility?
Summarize diffs and flag likely correctness, testing, and maintainability concerns for human review.

**Interview version:** “My responsibility was to summarize diffs and flag likely correctness, testing, and maintainability concerns for human review.”

### A — Action: What did you actually build?
Parsed changed hunks, retrieved coding standards, prompted the model with bounded context, produced file/line-level findings, and added confidence/evidence fields.

**Interview version:** “Parsed changed hunks, retrieved coding standards, prompted the model with bounded context, produced file/line-level findings, and added confidence/evidence fields.”

### R — Result: What was the outcome?
Reviewers got a focused first pass while retaining final decision-making responsibility.

**Interview version:** “Reviewers got a focused first pass while retaining final decision-making responsibility.”

> **Important:** Do not invent numbers. If you have real metrics—latency, throughput, users, error reduction, time saved—add them here.

## Architecture

```text
PR Diff → Parser → Context Retrieval → LLM Review → Structured Findings → Human Reviewer
```

When explaining architecture, go left to right: **request → validation/auth → business logic → data/storage → async/downstream response**.

## End-to-End Flow

```text
Fetch diff → chunk by file/function → retrieve standards → analyze → dedupe findings → present evidence
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

**Question:** How do you stop noisy or incorrect comments?

**Strong answer direction:** Use confidence thresholds, evidence requirements, deduplication, rule-based checks, and human approval before posting comments.

A good challenge answer should cover: **problem → options considered → decision → implementation → result/learning**.

## Common Follow-Up Questions

- How do you chunk code?
- How do you evaluate precision?
- RAG vs fine-tuning?
- How do you handle secrets?
- How would you integrate with GitHub?

## 60-Second Answer Template

“Large pull requests are time-consuming to review and repetitive issues can be missed. My responsibility was to summarize diffs and flag likely correctness, testing, and maintainability concerns for human review. Technically, parsed changed hunks, retrieved coding standards, prompted the model with bounded context, produced file/line-level findings, and added confidence/evidence fields. One important challenge was: how do you stop noisy or incorrect comments? I handled it by use confidence thresholds, evidence requirements, deduplication, rule-based checks, and human approval before posting comments. Overall, reviewers got a focused first pass while retaining final decision-making responsibility.”

## Before You Claim This Project

Make sure you can open the code and explain the schema, one API end-to-end, one failure case, one design trade-off, and one improvement you would make next.
