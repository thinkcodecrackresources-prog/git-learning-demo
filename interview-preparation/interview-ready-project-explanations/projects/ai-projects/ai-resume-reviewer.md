# AI Resume Reviewer

> **Practice project:** Use this as a structure. Replace every detail with what you genuinely built and can defend in an interview.

## Tech Stack

**Python, FastAPI, LLM, Vector Search**

## 30-Second Interview Introduction

“I worked on a **AI Resume Reviewer**. Candidates need structured feedback on how well a resume aligns with a job description. My main responsibility was to **parse resume/jd, identify gaps, and generate explainable suggestions without inventing experience.** I used **Python, FastAPI, LLM, Vector Search**. The key outcome was that users received section-level feedback, missing-skill signals, and rewrite suggestions grounded in their resume.”

## STAR Explanation

### S — Situation: What problem were you solving?
Candidates need structured feedback on how well a resume aligns with a job description.

**Interview version:** “Candidates need structured feedback on how well a resume aligns with a job description.”

### T — Task: What was your responsibility?
Parse resume/JD, identify gaps, and generate explainable suggestions without inventing experience.

**Interview version:** “My responsibility was to parse resume/JD, identify gaps, and generate explainable suggestions without inventing experience.”

### A — Action: What did you actually build?
Built document parsing, section extraction, embeddings for semantic matching, structured prompts, scoring components, and guardrails against fabricated claims.

**Interview version:** “Built document parsing, section extraction, embeddings for semantic matching, structured prompts, scoring components, and guardrails against fabricated claims.”

### R — Result: What was the outcome?
Users received section-level feedback, missing-skill signals, and rewrite suggestions grounded in their resume.

**Interview version:** “Users received section-level feedback, missing-skill signals, and rewrite suggestions grounded in their resume.”

> **Important:** Do not invent numbers. If you have real metrics—latency, throughput, users, error reduction, time saved—add them here.

## Architecture

```text
UI → FastAPI → Parser → Matcher/Vector Search → LLM → Structured Feedback
```

When explaining architecture, go left to right: **request → validation/auth → business logic → data/storage → async/downstream response**.

## End-to-End Flow

```text
Upload resume + JD → parse → normalize → retrieve matching evidence → generate feedback → validate output
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

**Question:** How do you reduce hallucinations?

**Strong answer direction:** Constrain generation to retrieved resume/JD evidence, require structured output, and reject suggestions that introduce unsupported experience.

A good challenge answer should cover: **problem → options considered → decision → implementation → result/learning**.

## Common Follow-Up Questions

- Why RAG?
- How do embeddings help?
- How do you evaluate quality?
- How do you protect PII?
- How would you prevent fake resume claims?

## 60-Second Answer Template

“Candidates need structured feedback on how well a resume aligns with a job description. My responsibility was to parse resume/JD, identify gaps, and generate explainable suggestions without inventing experience. Technically, built document parsing, section extraction, embeddings for semantic matching, structured prompts, scoring components, and guardrails against fabricated claims. One important challenge was: how do you reduce hallucinations? I handled it by constrain generation to retrieved resume/JD evidence, require structured output, and reject suggestions that introduce unsupported experience. Overall, users received section-level feedback, missing-skill signals, and rewrite suggestions grounded in their resume.”

## Before You Claim This Project

Make sure you can open the code and explain the schema, one API end-to-end, one failure case, one design trade-off, and one improvement you would make next.
