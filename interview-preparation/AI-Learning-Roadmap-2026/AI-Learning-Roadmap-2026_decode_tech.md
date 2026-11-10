# If I Had to Learn AI in 2026 — The Practical Roadmap

> **Decode Tech | From understanding LLMs to building real AI applications**
>
> **Path:** Understand LLMs → Build with LLM APIs → Master RAG → Build AI Agents → Learn MCP → Ship a capstone.

This is a hands-on learning roadmap for software engineers, students, and curious builders. You do **not** need to train a foundation model to build useful AI products. You should understand what models do, learn to use them through APIs, evaluate their outputs, and build reliable systems around them.

**Estimated pace:** 10 weeks at ~1–2 hours/day, assuming basic programming knowledge. Adjust freely; the stages overlap.

## Prerequisites and setup

- **Python:** functions, classes, exceptions, virtual environments, package management, HTTP requests, JSON, and basic async programming.
- **Developer tools:** Git/GitHub, terminal, VS Code or equivalent, API testing, environment variables.
- **Helpful foundations:** basic probability, vectors/dot products, SQL, HTTP, and API security.
- **Security from day one:** never commit API keys, personal documents, credentials, or private data to GitHub. Use `.env` locally and a secret manager in deployment.
- **Budget:** most learning resources below are free to read. Hosted model APIs, SMS/search services, and cloud hosting may cost money. Set usage caps and monitor spending.

### Suggested repo layout

```text
ai-learning-2026/
├── 01-llm-fundamentals/
├── 02-llm-apis/
├── 03-rag/
├── 04-ai-agents/
├── 05-mcp/
├── 06-capstone/
├── notes/
└── README.md
```

---

## Stage 1 — Understand How LLMs Work (Weeks 1–2)

**Goal:** Explain what happens between a user prompt and a model response, including the model's limitations.

### Learn these concepts

1. **Tokens and tokenization:** why text becomes token IDs; tokens are not always words; why token counts affect cost and context.
2. **Embeddings:** numerical vectors representing text or other inputs; similarity and retrieval use cases.
3. **Transformers:** token embeddings, positional information, stacked transformer blocks, and next-token prediction in autoregressive LLMs.
4. **Attention:** how token representations incorporate information from other tokens; conceptual roles of queries, keys, and values.
5. **Context window:** how much input and generated output can fit within a model's context limit; truncation and long-context trade-offs.
6. **Temperature and sampling:** how generation settings can affect variability; behavior depends on model and API support.
7. **Inference:** prompt processing, token generation, latency, and why output can be wrong despite sounding convincing.
8. **Training vs. fine-tuning vs. inference:** know the difference and when each matters.

### Resources

| Resource | What it is useful for |
|---|---|
| [Hugging Face LLM Course](https://huggingface.co/learn/llm-course/chapter1/1) | Free introduction to transformers, tokenization, and NLP workflows. |
| [The Illustrated Transformer — Jay Alammar](https://jalammar.github.io/illustrated-transformer/) | Visual intuition for transformer internals. |
| [Let's build GPT — Andrej Karpathy](https://www.youtube.com/watch?v=kCc8FmEb1nY) | Hands-on, code-level explanation of a small GPT-style model. |
| [nanoGPT](https://github.com/karpathy/nanoGPT) | Small model-training codebase to inspect after learning the basics. |
| [Hugging Face Transformers Docs](https://huggingface.co/docs/transformers/index) | Practical model and tokenizer APIs. |

### Practice

- Tokenize five sentences, including code, punctuation, and non-English text. Compare token counts.
- Explain in your own words why the same word may have different contextual representations.
- Draw: **prompt → tokenizer → model → next-token probabilities → generated tokens → response**.
- Compare a deterministic-looking output setting with a more variable setting, if supported by your chosen model.

### Mini-project: Tokenizer Explorer

**Input:** Text entered by a user. **Output:** token strings/IDs, token count, and a visual token breakdown.

**Implementation outline:**
1. Create a simple Streamlit interface.
2. Use a tokenizer from Hugging Face or a tokenizer matching your selected API model.
3. Display the tokens and count, with a warning that tokenization varies by model.
4. Add examples for code, emojis, and multilingual input.
5. Write a README with screenshots and findings.

**Done when:** You can explain why token counts differ and demonstrate the tool without using a paid API.

---

## Stage 2 — Learn to Build with LLM APIs (Weeks 3–4)

**Goal:** Build an application that calls a model, validates outputs, handles errors, and tracks cost.

### Learn these concepts

1. **API basics:** authentication, request/response structure, model identifiers, timeouts, retries, and rate limits.
2. **Prompt design:** clear instructions, context, examples, and explicit output requirements; evaluate rather than assume reliability.
3. **Structured outputs:** use schema-based responses where supported; validate parsed data before use.
4. **Tool/function calling:** the model proposes tool arguments; **your application** validates permissions, executes tools, and returns results.
5. **Streaming:** display partial output while generation is underway; handle interruptions and errors.
6. **Model selection:** quality, latency, token/context limits, supported features, and cost.
7. **Safety:** prompt injection, untrusted retrieved content, PII handling, and human approval for consequential actions.
8. **Evaluation:** build a small test set and measure correctness, consistency, and failure modes.

### Resources

| Resource | What it is useful for |
|---|---|
| [OpenAI API Docs](https://developers.openai.com/api/docs) | Official API concepts and current model capabilities. |
| [OpenAI Structured Outputs](https://developers.openai.com/api/docs/guides/structured-outputs) | Schema-conforming model outputs. |
| [OpenAI Function Calling](https://developers.openai.com/api/docs/guides/function-calling) | Model-to-tool integration patterns. |
| [OpenAI Cookbook](https://github.com/openai/openai-cookbook) | Practical code examples and implementation patterns. |
| [Anthropic Developer Docs](https://docs.anthropic.com/) | Alternative provider APIs and tool-use guidance. |
| [Google Gemini API Docs](https://ai.google.dev/gemini-api/docs) | Alternative provider API documentation. |

### Example architecture

```text
Web UI → Your Backend API → LLM Provider
            ↓                  ↓
       Input validation     Model response
            ↓                  ↓
       Logging/metrics ← Output validation
            ↓
       Return result to UI
```

### Practice

- Create an endpoint that accepts a short text and returns a structured summary.
- Reject missing/invalid input; never expose provider credentials to the browser.
- Add request timeout, bounded retry with backoff for transient failures, and clear error responses.
- Record request latency and token usage where available; avoid logging sensitive prompts by default.
- Test at least 10 representative inputs, including empty, long, ambiguous, and adversarial ones.

### Mini-project: AI Resume Analyzer

**Input:** Resume text supplied with permission. **Output:** structured skills, experience highlights, suggested improvements, and uncertainty notes.

**Implementation outline:**
1. Build a FastAPI endpoint `POST /analyze`.
2. Validate file type, length, and text extraction if accepting uploads.
3. Ask the model for a structured response: `skills`, `experience_summary`, `improvements`, `warnings`.
4. Validate the response against a schema; handle refusals and malformed outputs.
5. Show suggestions without inventing qualifications or employment history.
6. Add a simple UI, tests, latency measurement, and an estimated cost per run.

**Done when:** The app handles malformed inputs safely and outputs consistent, validated data.

---

## Stage 3 — Master RAG (Weeks 5–6)

**Goal:** Answer questions using an external knowledge collection, with source citations and measurable retrieval quality.

### What RAG means

**Retrieval-Augmented Generation** retrieves relevant material from an external source and includes it in the model's context. It can improve grounding, but does **not** guarantee correctness or eliminate hallucinations.

### Learn these concepts

1. **Ingestion:** extract text from documents; retain document IDs, page numbers, and access metadata.
2. **Chunking:** choose chunk size, overlap, and structure-aware splitting.
3. **Embeddings and vector stores:** turn chunks into vectors and search by similarity.
4. **Retrieval:** top-k search, metadata filters, and access-controlled search.
5. **Hybrid search:** combine keyword and semantic retrieval when appropriate.
6. **Reranking:** reorder retrieved candidates using a more precise relevance model.
7. **Prompt augmentation:** pass selected excerpts and citations to the model; instruct it to acknowledge missing evidence.
8. **Evaluation:** retrieval recall/precision, answer correctness, citation faithfulness, latency, and cost.
9. **Freshness:** document updates, re-indexing, deletion, and stale content handling.

### Resources

| Resource | What it is useful for |
|---|---|
| [LangChain Python Docs](https://docs.langchain.com/oss/python/langchain/overview) | Retrieval pipelines and integrations. |
| [LlamaIndex Docs](https://docs.llamaindex.ai/) | Data ingestion, indexing, and document Q&A. |
| [Chroma Docs](https://docs.trychroma.com/) | Local vector store and similarity search. |
| [Pinecone RAG Guide](https://www.pinecone.io/learn/retrieval-augmented-generation/) | RAG concepts and design decisions. |
| [FAISS](https://github.com/facebookresearch/faiss) | Vector similarity search library. |

### Typical RAG flow

```text
INGESTION (offline)
Documents → Parse → Chunk → Embed → Vector Store + Metadata

QUESTION ANSWERING (online)
Question → Embed/Keyword Search → Retrieve → (Rerank)
         → Build prompt with citations → LLM → Answer + Sources
```

### Mini-project: PDF Chatbot

**Implementation outline:**
1. Upload 3–10 non-sensitive PDFs.
2. Extract text and preserve document/page references.
3. Chunk and embed the text; store vectors plus metadata.
4. Retrieve top-k chunks for each question.
5. Ask the LLM to answer only using retrieved evidence and cite the source pages.
6. Add an explicit “I couldn't find enough evidence” response.
7. Create 20 test questions: easy, multi-document, ambiguous, and unanswerable.
8. Track retrieval hit rate, grounded answer quality, and citation accuracy.

**Done when:** Users can inspect the source passages supporting answers, and the app declines unsupported questions.

### Common RAG mistakes

- Chunking without preserving context or source IDs.
- Using vector similarity alone for exact IDs, names, or error codes.
- Treating retrieved documents as trusted instructions.
- Indexing private content without enforcing per-user access controls.
- Measuring only how fluent the answer sounds, not whether the citations support it.

---

## Stage 4 — Move to AI Agents (Weeks 7–8)

**Goal:** Build a tool-using assistant that completes multi-step tasks with controlled permissions and observable execution.

### Learn these concepts

1. **Agent vs. chatbot:** an agent can select tools and iterate toward a goal; not every workflow needs an agent.
2. **Tool calling:** define narrow, typed tool contracts and validate arguments.
3. **Agent loop:** goal → choose action → execute tool → observe result → continue or stop.
4. **Planning:** break down tasks; limit steps, time, and spending.
5. **Memory/state:** store only useful state; distinguish current context from persistent user data.
6. **Multi-agent workflows:** separate roles only when simpler orchestration is insufficient.
7. **Human-in-the-loop:** require approval before modifying files, publishing, spending money, or other consequential actions.
8. **Reliability:** retries, idempotency, audit logs, tool failures, and graceful termination.
9. **Security:** least-privilege credentials, prompt-injection defenses, sandboxing, and tool-result validation.

### Resources

| Resource | What it is useful for |
|---|---|
| [Hugging Face Agents Course](https://huggingface.co/learn/agents-course/unit0/introduction) | Agent concepts and hands-on exercises. |
| [LangGraph Docs](https://docs.langchain.com/oss/python/langgraph/overview) | Stateful workflows and agent orchestration. |
| [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/) | Agent orchestration in Python. |
| [LangGraph Repository](https://github.com/langchain-ai/langgraph) | Source and examples. |

### Example agent loop

```text
User Goal
   ↓
Agent chooses a permitted tool
   ↓
Validate tool inputs + permissions
   ↓
Execute tool → Observe result
   ↓
Continue? ── Yes → Next tool/step
   │
   No
   ↓
Final answer + action summary
```

### Mini-project: GitHub Repository Assistant

**Features:** list files, search code, read selected files, summarize architecture, identify possible improvements.

**Implementation outline:**
1. Connect to a local demo repository with read-only access.
2. Expose `list_files`, `search_code`, and `read_file` tools with safe path validation.
3. Set maximum tool calls and time limits.
4. Ask the agent to answer questions with file paths and line references.
5. Treat repository text as untrusted; ignore instructions embedded in code comments.
6. If adding a code-edit tool, make it proposal-only until a human reviews and approves changes.
7. Test nonexistent files, large repositories, injected instructions, and tool errors.

**Done when:** The assistant can explain repository structure and show evidence without making unauthorized changes.

---

## Stage 5 — Learn MCP (Week 9)

**Goal:** Connect an AI host to external tools and data using a standard protocol, while preserving access controls.

### What MCP is

**Model Context Protocol (MCP)** is a protocol for connecting AI applications to tools, resources, and reusable prompts. It is **not** itself a model or an autonomous agent. An MCP host manages clients, and clients communicate with MCP servers that expose capabilities.

### Learn these concepts

1. **Host, client, server:** understand who initiates connections and who exposes capabilities.
2. **Tools:** callable operations with structured inputs and outputs.
3. **Resources:** data made available to the host/client.
4. **Prompts:** reusable prompt templates exposed by a server.
5. **Transport and lifecycle:** connection setup, capability discovery, requests, and responses.
6. **Security:** authentication where applicable, user consent, permission scopes, and untrusted server output.
7. **Operational design:** timeouts, logs, tool errors, and rate limiting.

### Resources

| Resource | What it is useful for |
|---|---|
| [Official MCP Documentation](https://modelcontextprotocol.io/docs) | Protocol concepts, architecture, and guides. |
| [MCP Introduction](https://modelcontextprotocol.io/introduction) | Beginner overview; follow the current docs if this path redirects. |
| [MCP Example Servers](https://github.com/modelcontextprotocol/servers) | Reference implementations and examples. |
| [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk) | Build an MCP server in Python. |
| [MCP Specification](https://modelcontextprotocol.io/specification) | Authoritative protocol details. |

### Architecture

```text
User → AI Host / Agent
           │
        MCP Client
           │
        MCP Server
           ├── Tool: search_documents(query)
           ├── Tool: get_record(id)
           ├── Resource: reference documents
           └── Prompt: summarize_findings
```

### Mini-project: Read-Only Knowledge MCP Server

**Implementation outline:**
1. Create a small SQLite database containing non-sensitive sample records.
2. Implement an MCP server using the official Python SDK.
3. Expose a `search_records(query, limit)` tool with validated parameters.
4. Expose a read-only resource or prompt template if useful.
5. Connect through a compatible MCP host/client.
6. Verify the host can discover the tool, call it, and present results accurately.
7. Enforce read-only access, query limits, and safe error handling.

**Done when:** A compatible AI host can query the sample data through MCP without direct database credentials in the model prompt.

---

## Week 10 — Capstone: Production-Minded AI Knowledge Assistant

**Build:** An assistant that answers questions about a small, authorized document collection and can query a read-only MCP tool.

### Features

- Document upload/ingestion with chunking, embeddings, and citations.
- RAG-based Q&A with a clear “insufficient evidence” fallback.
- Agent workflow that chooses between document retrieval and a read-only structured-data tool.
- MCP integration for the structured-data tool.
- Backend API and lightweight frontend.
- Observability: latency, errors, tool calls, token usage/cost, and trace IDs.
- Evaluation suite covering groundedness, retrieval, permission checks, and failure handling.

### Suggested architecture

```text
Browser UI
    ↓
FastAPI Backend ── Authentication / Rate Limits
    ↓
Agent Orchestrator ── LLM API
    ├── RAG Retriever → Vector Store ← Document Ingestion
    └── MCP Client → MCP Server → Read-only Sample Database
    ↓
Answer + Citations + Tool Activity Summary
    ↓
Logs / Traces / Evaluation Metrics
```

### Build checklist

- [ ] Store secrets outside source control.
- [ ] Validate inputs and file sizes.
- [ ] Enforce user/document access controls.
- [ ] Bound model and tool-call costs.
- [ ] Add timeouts, retries, and useful error messages.
- [ ] Treat retrieved text and tool output as untrusted.
- [ ] Include source references in grounded answers.
- [ ] Require approval for any future write-capable tools.
- [ ] Evaluate with at least 20 realistic questions and negative tests.
- [ ] Document architecture, setup, limitations, and demo screenshots.

### Evaluation rubric

| Dimension | How to test |
|---|---|
| Correctness | Compare answers with verified reference answers. |
| Grounding | Confirm cited passages actually support claims. |
| Retrieval | Measure whether relevant chunks appear in top-k results. |
| Safety | Try prompt injection, private-file requests, and invalid tool inputs. |
| Reliability | Simulate API timeouts, empty results, and unavailable tools. |
| Performance | Track p50/p95 response latency and cost per request. |

---

## Complete 10-Week Study Plan

| Week | Main topic | Practical deliverable |
|---|---|---|
| 1 | Tokens, embeddings, transformer intuition | Illustrated notes and tokenizer experiments |
| 2 | Attention, context, sampling, inference | Tokenizer Explorer |
| 3 | Model APIs, prompting, error handling | Working LLM API endpoint |
| 4 | Structured outputs, tool calls, streaming | AI Resume Analyzer |
| 5 | Chunking, embeddings, vector stores | Document ingestion and retrieval |
| 6 | Hybrid retrieval, reranking, evaluation | PDF Chatbot with citations |
| 7 | Agent loops, tools, state | Read-only repository tools |
| 8 | Agent safety, limits, observability | GitHub Repository Assistant |
| 9 | MCP hosts, clients, servers, tools | Read-only MCP Server |
| 10 | Integrate, evaluate, deploy | AI Knowledge Assistant |

## What to Learn Next (Optional)

- **Fine-tuning:** when prompting and RAG are not sufficient; dataset quality and evaluation.
- **Multimodal AI:** image, audio, and video inputs/outputs.
- **Model serving:** quantization, batching, throughput, GPU memory, and local inference.
- **AI system design:** caching, queues, async jobs, rate limits, multi-tenancy, cost controls.
- **Evaluation and governance:** bias, privacy, security reviews, and regression testing.

## Frequently Asked Questions

**Do I need advanced math?** Not to start building LLM applications. Linear algebra, probability, and optimization become more important for model internals and training.

**Do I need a GPU?** No for most API-based projects. Some local models and training exercises benefit from one.

**Do I need LangChain?** No. Start with plain API calls; add a framework when orchestration complexity justifies it.

**Is RAG the same as fine-tuning?** No. RAG supplies external context at query time; fine-tuning changes model parameters through training.

**Are AI agents always better than workflows?** No. Prefer deterministic workflows for predictable tasks. Use agents when dynamic tool selection adds value.

**Is MCP required to build agents?** No. MCP standardizes certain tool/data connections; agents can use other integrations.

**Can I finish in 10 weeks?** You can build a solid foundation and several prototypes, but expertise requires continued practice, debugging, and evaluation.

## Share This Guide

**Decode Tech — Concepts Every Software Engineer Should Know**

If this roadmap helped, save it, share it with another developer, and build at least one project per stage.

