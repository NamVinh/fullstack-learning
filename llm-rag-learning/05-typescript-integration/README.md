# Phase 05 — TypeScript Integration

> Biến lý thuyết từ Phase 01–04 thành code TypeScript chạy được trong NestJS backend
> và Next.js frontend. Đây là phase thực hành — code trực tiếp.

---

## 1. Setup Project (NestJS + AI)

```bash
# Existing NestJS project — thêm dependencies
npm install @anthropic-ai/sdk openai langchain @langchain/openai @langchain/community
npm install @qdrant/js-client-rest  # hoặc pgvector
npm install pdf-parse mammoth       # document parsing
npm install ai @ai-sdk/anthropic    # Vercel AI SDK cho streaming
```

```typescript
// Environment variables
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...
QDRANT_URL=http://localhost:6333
DATABASE_URL=postgresql://...
```

---

## 2. NestJS AI Service — Pattern đầy đủ

### Module structure

```
src/
  ai/
    ai.module.ts
    services/
      llm.service.ts        ← wrapper quanh Anthropic/OpenAI SDK
      embedding.service.ts  ← embed text thành vectors
      rag.service.ts        ← orchestrate retrieval + generation
    vector-store/
      vector-store.interface.ts
      pgvector.service.ts   ← pgvector implementation
      qdrant.service.ts     ← Qdrant implementation
    document-processor/
      document-processor.service.ts  ← load, chunk, index documents
```

### LLM Service

```typescript
// ai/services/llm.service.ts
import { Injectable } from "@nestjs/common";
import Anthropic from "@anthropic-ai/sdk";

export interface LLMMessage {
  role: "user" | "assistant";
  content: string;
}

export interface LLMOptions {
  temperature?: number;
  maxTokens?: number;
  system?: string;
}

@Injectable()
export class LLMService {
  private client: Anthropic;

  constructor() {
    this.client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });
  }

  async complete(
    messages: LLMMessage[],
    options: LLMOptions = {}
  ): Promise<string> {
    const response = await this.client.messages.create({
      model: "claude-sonnet-4-6",
      max_tokens: options.maxTokens ?? 2048,
      temperature: options.temperature ?? 0,
      system: options.system,
      messages,
    });

    return response.content[0].type === "text" ? response.content[0].text : "";
  }

  async completeWithTools<T>(
    messages: LLMMessage[],
    toolName: string,
    toolSchema: Anthropic.Tool["input_schema"],
    options: LLMOptions = {}
  ): Promise<T> {
    const response = await this.client.messages.create({
      model: "claude-sonnet-4-6",
      max_tokens: options.maxTokens ?? 2048,
      temperature: 0,  // structured output — always 0
      system: options.system,
      tools: [{ name: toolName, description: toolName, input_schema: toolSchema }],
      tool_choice: { type: "tool", name: toolName },
      messages,
    });

    const toolUse = response.content.find((b) => b.type === "tool_use");
    if (!toolUse || toolUse.type !== "tool_use") {
      throw new Error("No tool use in response");
    }

    return toolUse.input as T;
  }

  // Streaming — returns AsyncIterable
  async *stream(
    messages: LLMMessage[],
    options: LLMOptions = {}
  ): AsyncIterable<string> {
    const stream = this.client.messages.stream({
      model: "claude-sonnet-4-6",
      max_tokens: options.maxTokens ?? 2048,
      system: options.system,
      messages,
    });

    for await (const chunk of stream) {
      if (
        chunk.type === "content_block_delta" &&
        chunk.delta.type === "text_delta"
      ) {
        yield chunk.delta.text;
      }
    }
  }
}
```

### Embedding Service

```typescript
// ai/services/embedding.service.ts
import { Injectable } from "@nestjs/common";
import OpenAI from "openai";

@Injectable()
export class EmbeddingService {
  private client: OpenAI;
  private model = "text-embedding-3-small";

  constructor() {
    this.client = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });
  }

  async embed(text: string): Promise<number[]> {
    const response = await this.client.embeddings.create({
      model: this.model,
      input: text,
    });
    return response.data[0].embedding;
  }

  async embedBatch(texts: string[]): Promise<number[][]> {
    // API cho phép batch tối đa 2048 inputs
    const batchSize = 100;
    const results: number[][] = [];

    for (let i = 0; i < texts.length; i += batchSize) {
      const batch = texts.slice(i, i + batchSize);
      const response = await this.client.embeddings.create({
        model: this.model,
        input: batch,
      });
      results.push(...response.data.map((d) => d.embedding));
    }

    return results;
  }
}
```

### RAG Service

```typescript
// ai/services/rag.service.ts
import { Injectable, Inject } from "@nestjs/common";
import { LLMService } from "./llm.service";
import { EmbeddingService } from "./embedding.service";
import { VectorStore, SearchResult } from "../vector-store/vector-store.interface";

interface RAGOptions {
  topK?: number;
  filter?: Record<string, any>;
  systemPrompt?: string;
}

@Injectable()
export class RAGService {
  constructor(
    private llm: LLMService,
    private embedding: EmbeddingService,
    @Inject("VECTOR_STORE") private vectorStore: VectorStore
  ) {}

  async query(userQuery: string, options: RAGOptions = {}): Promise<string> {
    const { topK = 5, filter, systemPrompt } = options;

    // 1. Retrieve
    const queryEmbedding = await this.embedding.embed(userQuery);
    const chunks = await this.vectorStore.search(queryEmbedding, topK, filter);

    if (chunks.length === 0) {
      return "I could not find relevant information to answer your question.";
    }

    // 2. Format context
    const context = this.formatContext(chunks);

    // 3. Generate
    const system =
      systemPrompt ??
      `You are an insurance assistant. Answer based ONLY on the context below.
If the answer is not in the context, say so clearly. Cite [Source N] when referencing.

CONTEXT:
${context}`;

    return this.llm.complete(
      [{ role: "user", content: userQuery }],
      { system, temperature: 0 }
    );
  }

  private formatContext(chunks: SearchResult[]): string {
    return chunks
      .map((c, i) => `[Source ${i + 1}]\n${c.content}`)
      .join("\n\n---\n\n");
  }
}
```

---

## 3. Document Processing Service

```typescript
// ai/document-processor/document-processor.service.ts
import { Injectable } from "@nestjs/common";
import { RecursiveCharacterTextSplitter } from "langchain/text_splitter";
import pdf from "pdf-parse";
import { EmbeddingService } from "../services/embedding.service";
import { VectorStore } from "../vector-store/vector-store.interface";
import { Inject } from "@nestjs/common";

export interface DocumentMetadata {
  documentId: string;
  documentType: "policy" | "claim" | "underwriting";
  policyNumber?: string;
  customerId?: string;
  fileName: string;
}

@Injectable()
export class DocumentProcessorService {
  private splitter: RecursiveCharacterTextSplitter;

  constructor(
    private embedding: EmbeddingService,
    @Inject("VECTOR_STORE") private vectorStore: VectorStore
  ) {
    this.splitter = new RecursiveCharacterTextSplitter({
      chunkSize: 1000,
      chunkOverlap: 200,
    });
  }

  async indexPDF(buffer: Buffer, metadata: DocumentMetadata): Promise<void> {
    // 1. Parse PDF
    const { text } = await pdf(buffer);

    // 2. Chunk
    const chunks = await this.splitter.splitText(text);

    // 3. Embed
    const embeddings = await this.embedding.embedBatch(chunks);

    // 4. Store
    await this.vectorStore.upsert(
      chunks.map((content, i) => ({
        id: `${metadata.documentId}_${i}`,
        content,
        embedding: embeddings[i],
        metadata: { ...metadata, chunkIndex: i },
      }))
    );
  }

  async deleteDocument(documentId: string): Promise<void> {
    await this.vectorStore.deleteByDocumentId(documentId);
  }
}
```

---

## 4. Controller — REST API endpoint

```typescript
// ai/ai.controller.ts
import { Controller, Post, Body, Get, Query, UploadedFile, UseInterceptors } from "@nestjs/common";
import { FileInterceptor } from "@nestjs/platform-express";
import { RAGService } from "./services/rag.service";
import { DocumentProcessorService } from "./document-processor/document-processor.service";

@Controller("ai")
export class AIController {
  constructor(
    private rag: RAGService,
    private docProcessor: DocumentProcessorService
  ) {}

  @Post("query")
  async query(@Body() body: { question: string; policyNumber?: string }) {
    const answer = await this.rag.query(body.question, {
      filter: body.policyNumber
        ? { policyNumber: body.policyNumber }
        : undefined,
    });
    return { answer };
  }

  @Post("documents/upload")
  @UseInterceptors(FileInterceptor("file"))
  async uploadDocument(
    @UploadedFile() file: Express.Multer.File,
    @Body() body: { documentType: string; policyNumber: string; customerId: string }
  ) {
    await this.docProcessor.indexPDF(file.buffer, {
      documentId: `${body.policyNumber}_${Date.now()}`,
      documentType: body.documentType as any,
      policyNumber: body.policyNumber,
      customerId: body.customerId,
      fileName: file.originalname,
    });
    return { success: true, message: "Document indexed successfully" };
  }
}
```

---

## 5. Frontend — Vercel AI SDK với Next.js (bạn đã biết Next.js)

### API Route (App Router)

```typescript
// app/api/chat/route.ts
import { streamText } from "ai";
import { anthropic } from "@ai-sdk/anthropic";

export async function POST(req: Request) {
  const { messages, policyNumber } = await req.json();

  // Fetch context từ NestJS backend
  const contextRes = await fetch(`${process.env.API_URL}/ai/retrieve`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      query: messages[messages.length - 1].content,
      policyNumber,
    }),
  });
  const { context } = await contextRes.json();

  const result = streamText({
    model: anthropic("claude-sonnet-4-6"),
    system: `You are an insurance assistant. Answer based on this context:
${context}`,
    messages,
  });

  return result.toDataStreamResponse();
}
```

### Chat UI Component (bạn đã biết React)

```typescript
// components/InsuranceChatbot.tsx
"use client";
import { useChat } from "ai/react";

export function InsuranceChatbot({ policyNumber }: { policyNumber?: string }) {
  const { messages, input, handleInputChange, handleSubmit, isLoading } = useChat({
    api: "/api/chat",
    body: { policyNumber },  // sent với mỗi request
  });

  return (
    <div className="flex flex-col h-96 border rounded-lg">
      <div className="flex-1 overflow-y-auto p-4 space-y-4">
        {messages.map((m) => (
          <div key={m.id} className={`flex ${m.role === "user" ? "justify-end" : "justify-start"}`}>
            <div className={`rounded-lg p-3 max-w-xs ${m.role === "user" ? "bg-blue-500 text-white" : "bg-gray-100"}`}>
              {m.content}
            </div>
          </div>
        ))}
        {isLoading && <div className="text-gray-400 text-sm">Analyzing...</div>}
      </div>
      <form onSubmit={handleSubmit} className="border-t p-4 flex gap-2">
        <input
          value={input}
          onChange={handleInputChange}
          placeholder="Ask about your policy..."
          className="flex-1 border rounded px-3 py-2 text-sm"
        />
        <button type="submit" disabled={isLoading} className="bg-blue-500 text-white px-4 py-2 rounded text-sm">
          Send
        </button>
      </form>
    </div>
  );
}
```

---

## 6. LangChain.js — Khi nào dùng

```typescript
// LangChain có nhiều built-in components — dùng khi không muốn tự build

import { ChatAnthropic } from "@langchain/anthropic";
import { MemoryVectorStore } from "langchain/vectorstores/memory";
import { OpenAIEmbeddings } from "@langchain/openai";
import { createRetrievalChain } from "langchain/chains/retrieval";
import { createStuffDocumentsChain } from "langchain/chains/combine_documents";
import { ChatPromptTemplate } from "@langchain/core/prompts";
import { RecursiveCharacterTextSplitter } from "langchain/text_splitter";
import { PDFLoader } from "@langchain/community/document_loaders/fs/pdf";

// Full RAG pipeline với LangChain — ít code hơn tự build
async function buildRAGChain(pdfPath: string) {
  // 1. Load
  const loader = new PDFLoader(pdfPath);
  const docs = await loader.load();

  // 2. Split
  const splitter = new RecursiveCharacterTextSplitter({
    chunkSize: 1000,
    chunkOverlap: 200,
  });
  const chunks = await splitter.splitDocuments(docs);

  // 3. Embed + Store (in-memory for demo)
  const vectorStore = await MemoryVectorStore.fromDocuments(
    chunks,
    new OpenAIEmbeddings({ model: "text-embedding-3-small" })
  );

  // 4. Build chain
  const llm = new ChatAnthropic({ model: "claude-sonnet-4-6" });
  const retriever = vectorStore.asRetriever(5);

  const prompt = ChatPromptTemplate.fromTemplate(`
Answer based ONLY on the context:
{context}

Question: {input}
`);

  const documentChain = await createStuffDocumentsChain({ llm, prompt });
  const retrievalChain = await createRetrievalChain({ retriever, combineDocsChain: documentChain });

  return retrievalChain;
}

// Usage
const chain = await buildRAGChain("policy.pdf");
const result = await chain.invoke({ input: "What does my policy cover?" });
console.log(result.answer);
```

**Dùng LangChain khi:** Prototype nhanh, cần nhiều built-in connectors (PDF, web, database loaders).  
**Tự build khi:** Cần full control, production performance, ít dependencies.

---

## Câu hỏi phỏng vấn

**Q: Làm sao handle error khi LLM API timeout hoặc rate limit?**
A: Exponential backoff retry, circuit breaker pattern (dùng `p-retry` hoặc NestJS `@nestjs/throttler`). Queue requests khi rate limit hit. Fallback to cheaper/faster model (Haiku thay Sonnet) nếu cần. Log mọi LLM call với input token count để monitor cost và detect anomaly.

**Q: Làm sao test code integrate với LLM?**
A: Unit test: mock LLM service (`jest.mock`), test business logic riêng. Integration test: dùng real API với fixture inputs và snapshot test output (không expect exact string vì LLM output không deterministic — test output matches schema, not exact content). E2E: test golden path với known questions và expected answer range.
