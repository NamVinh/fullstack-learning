# LLM / RAG / GenAI Engineering — Learning Roadmap

> Mục tiêu: Hiểu đủ để BUILD sản phẩm GenAI (không phải train model), tự tin
> phỏng vấn tại các công ty InsurTech / FinTech / Enterprise AI, và contribute
> vào hệ thống RAG pipeline, AI agent, LLM integration bằng TypeScript.

---

## Tại sao học cái này?

JD InsurTech yêu cầu:
- Build **GenAI-powered enterprise system** (Claim, Underwriting, FWA detection)
- TypeScript cho cả frontend và backend
- Kinh nghiệm dùng **Claude Code / Codex / Cursor Agent** (preferred)

Bạn đã có phần AI tool usage. Phần còn thiếu là biết **build sản phẩm tích hợp LLM**
— RAG pipeline, agent, structured output, document processing. Đây là nội dung folder này.

---

## Roadmap — 13 Phase (đầy đủ)

| Phase | Chủ đề | Nội dung chính | Thời gian |
|-------|--------|----------------|-----------|
| **01** | LLM Basics | Transformer, tokenization, context window, temperature, models, cost, streaming, multimodal | 2–3 ngày |
| **02** | Prompt Engineering | Zero/few-shot, CoT, structured output (JSON + tool calling), Zod, security, templating | 3–4 ngày |
| **03** | RAG Architecture | Full pipeline, chunking strategies, embedding, retrieval, advanced patterns (HyDE, multi-query) | 1 tuần |
| **04** | Vector Databases | pgvector, Qdrant, ChromaDB — setup, CRUD, filtering, NestJS abstraction | 3–4 ngày |
| **05** | TypeScript Integration | NestJS service patterns, LangChain.js, Vercel AI SDK, streaming in Next.js | 1 tuần |
| **06** | Agentic AI | Tool calling, ReAct agent loop, memory, human-in-the-loop, multi-agent | 1 tuần |
| **07** | Production Patterns | Evaluation, observability, semantic caching, prompt caching, guardrails, cost control | 1 tuần |
| **08** | InsurTech Context | FWA engine, document extraction, underwriting AI, compliance, interview Q&A | 3–4 ngày |
| **09** | Fine-Tuning | Khi nào fine-tune, LoRA/QLoRA, OpenAI fine-tuning API, data preparation | 3–4 ngày |
| **10** | Multimodal & OCR | PDF processing, AWS Textract, Google DocAI, Azure Doc Intelligence, LLM vision | 3–4 ngày |
| **11** | Advanced Retrieval | GraphRAG, reranking, context compression, self-RAG, CRAG, parent retrieval | 1 tuần |
| **12** | Evaluation (RAGAS) | 4 core metrics, golden dataset, LLM-as-judge, CI/CD integration, A/B testing | 3–4 ngày |
| **13** | Security & Compliance | Prompt injection, PII handling, data isolation, GDPR, AI explainability, audit trail | 3–4 ngày |

**Tổng ước tính: 8–10 tuần học nghiêm túc (2–3 giờ/ngày)**

---

## Thứ tự học — Critical Path

```
FOUNDATION (học trước, mọi thứ phụ thuộc vào đây):
  01 LLM Basics → 02 Prompt Engineering

CORE SKILLS (core của mọi GenAI engineer):
  03 RAG Architecture → 04 Vector Databases → 05 TypeScript Integration

ADVANCED (sau khi đã solid core):
  06 Agentic AI → 10 Multimodal & OCR → 11 Advanced Retrieval

DOMAIN (apply vào InsurTech):
  08 InsurTech Context  ← đọc sớm để có context khi học core

PRODUCTION-READY:
  07 Production Patterns → 12 Evaluation → 13 Security & Compliance

SPECIALIZED (khi cần):
  09 Fine-Tuning  ← chỉ khi RAG không đủ
```

---

## Stack chính bạn sẽ dùng

```
Language:           TypeScript (Node.js) — bạn đã biết
Backend framework:  NestJS — bạn đã biết
Frontend:           Next.js + Vercel AI SDK — bạn đã biết

LLM Providers:
  Anthropic Claude  @anthropic-ai/sdk
  OpenAI GPT-4o     openai

Vector Databases:
  pgvector          PostgreSQL extension — bạn đã biết PostgreSQL
  Qdrant            Dedicated vector DB (open source)

Embedding Models:
  text-embedding-3-small (OpenAI) — default
  voyage-3 (Anthropic) — higher quality

RAG Framework:
  LangChain.js      langchain, @langchain/openai, @langchain/anthropic
  Vercel AI SDK     ai, @ai-sdk/anthropic — best for Next.js streaming

Document Processing:
  pdf-parse         Text-based PDF
  AWS Textract      Tables, forms, scanned docs
  Azure Doc Intel   Pre-built insurance models

Evaluation:
  RAGAS             Python (pip install ragas)
  LLM-as-Judge      Inline evaluation với Claude

Observability:
  OpenTelemetry     Bạn đã biết từ Herond
  LangSmith         LangChain tracing (optional)
```

---

## Quick Reference — Câu hỏi phỏng vấn thường gặp

### LLM Fundamentals
- **LLM là gì?** → Hàm predict next token, không store facts, có thể hallucinate
- **RAG là gì?** → Retrieve relevant docs → inject context → generate grounded answer
- **Context window?** → Claude 200k tokens, GPT-4o 128k tokens
- **Khi nào hallucinate?** → Khi không có context, khi context mâu thuẫn, khi over-confident

### RAG Pipeline
- **Chunking strategy?** → Recursive character splitter (1000 chars, 200 overlap) — default. Semantic chunking khi cần accuracy.
- **Embedding model?** → text-embedding-3-small (OpenAI) default. Voyage-3 nếu cần tốt hơn.
- **pgvector vs Qdrant?** → pgvector nếu đã có PostgreSQL. Qdrant khi cần hybrid search hoặc scale.
- **Improve RAG?** → Add reranking (Cohere) → hybrid search → context compression → parent retrieval

### Production
- **Evaluate RAG?** → RAGAS 4 metrics: faithfulness, relevance, context precision, context recall
- **Reduce cost?** → Semantic caching, prompt caching, model routing (haiku for simple), batch embedding
- **Security?** → Structural separation, tool calling forced output, PII redaction, customer isolation

### InsurTech Specific
- **FWA engine?** → Rule engine (fast, free) + LLM analysis (nuanced) + human review (high-risk)
- **Insurance documents?** → pdf-parse (text PDF) → Textract (tables/forms) → vision model (scanned)
- **Explainability?** → Store reasoning + evidence sources + rule triggers in every AI decision

---

## Tài nguyên học

### Đọc / xem
- [Anthropic Cookbook](https://github.com/anthropics/anthropic-cookbook) — ví dụ thực tế
- [OpenAI Cookbook](https://cookbook.openai.com/) — patterns & techniques  
- [LangChain JS Docs](https://js.langchain.com/docs/) — framework
- [Vercel AI SDK](https://sdk.vercel.ai/docs) — streaming với Next.js
- [RAGAS Docs](https://docs.ragas.io/) — evaluation framework

### Build (quan trọng hơn đọc)
1. **Mini project 1:** Build policy Q&A chatbot — upload PDF policy → ask questions → get answers
2. **Mini project 2:** Build claim classifier — input claim description → output structured JSON
3. **Mini project 3:** Add FWA scoring — integrate rule engine + LLM analysis
4. **Mini project 4:** Add evaluation — golden dataset + RAGAS metrics

### Tools để practice ngay
- Claude API free tier: [console.anthropic.com](https://console.anthropic.com)
- OpenAI API: [platform.openai.com](https://platform.openai.com)
- Qdrant local: `docker run -p 6333:6333 qdrant/qdrant`
- pgvector: `docker run -e POSTGRES_PASSWORD=pass -p 5432:5432 pgvector/pgvector:pg16`
