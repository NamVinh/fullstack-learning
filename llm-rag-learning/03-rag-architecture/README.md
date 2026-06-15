# Phase 03 — RAG Architecture

> RAG (Retrieval-Augmented Generation) = cung cấp cho LLM thông tin cụ thể, up-to-date
> từ knowledge base của bạn, thay vì chỉ dựa vào knowledge được train sẵn.
> Đây là core pattern của hầu hết enterprise GenAI systems.

---

## 1. Tại sao cần RAG?

```
Vấn đề với LLM thuần:
- Không biết data nội bộ của công ty (policy documents, claim history)
- Training cutoff — không có thông tin mới
- Hallucinate khi không có data
- Không cite được source

RAG giải quyết:
- Retrieve relevant documents từ knowledge base trước
- Inject vào context của LLM
- LLM answer dựa trên document thực tế → ít hallucination hơn
- Có thể cite exact source
```

---

## 2. RAG Pipeline — Full Flow

```
                    INDEXING (offline, một lần)
                    ─────────────────────────────
  Documents ──► Load ──► Chunk ──► Embed ──► Store (Vector DB)


                    RETRIEVAL + GENERATION (online, mỗi query)
                    ───────────────────────────────────────────
  User Query ──► Embed Query ──► Vector Search ──► Top-K Chunks
                                                        │
                                              Inject into Prompt
                                                        │
                                               LLM generates ──► Response
```

---

## 3. Indexing Pipeline — chi tiết

### Step 1: Document Loading

```typescript
import fs from "fs";
import pdf from "pdf-parse";

// Load PDF (insurance policy document)
async function loadPDF(filePath: string): Promise<string> {
  const dataBuffer = fs.readFileSync(filePath);
  const data = await pdf(dataBuffer);
  return data.text;
}

// Load từ URL
async function loadURL(url: string): Promise<string> {
  const response = await fetch(url);
  return response.text();
}

// Load nhiều document types — trong practice dùng LangChain loaders
```

### Step 2: Chunking — quan trọng nhất

Chunking = chia document thành chunks nhỏ để embed và retrieve.

**Fixed-size chunking:**
```typescript
function chunkBySize(
  text: string,
  chunkSize: number = 500,  // characters
  overlap: number = 50       // overlap để không mất context ở boundary
): string[] {
  const chunks: string[] = [];
  let start = 0;

  while (start < text.length) {
    const end = Math.min(start + chunkSize, text.length);
    chunks.push(text.slice(start, end));
    start = end - overlap;  // overlap với chunk trước
  }

  return chunks;
}
```

**Recursive character chunking (tốt hơn — LangChain approach):**
```typescript
// Chia theo paragraph > sentence > word — tự nhiên hơn
import { RecursiveCharacterTextSplitter } from "langchain/text_splitter";

const splitter = new RecursiveCharacterTextSplitter({
  chunkSize: 1000,      // target size in characters
  chunkOverlap: 200,    // overlap
  separators: ["\n\n", "\n", ". ", " ", ""],  // priority order
});

const chunks = await splitter.splitText(documentText);
```

**Semantic chunking (tốt nhất — đắt hơn):**
```typescript
// Chia theo semantic boundary — dùng embedding similarity
// Chunk khi "topic thay đổi" thay vì theo fixed size
// LangChain: SemanticChunker — cần embedding model để detect boundaries
```

**Chunking strategies so sánh:**

| Strategy | Pros | Cons | Dùng khi |
|----------|------|------|----------|
| Fixed-size | Đơn giản, nhanh | Cắt giữa câu, mất context | Prototype |
| Recursive | Balance tốt | Còn phụ thuộc character count | Production default |
| Semantic | Preserve meaning | Chậm, tốn token | High-accuracy RAG |

**Insurance best practice:** Chunk theo section của document (Article 1, Section 2...) nếu document có structure rõ ràng.

---

### Step 3: Embedding

Embedding = chuyển text thành vector số — semantic similarity được capture.

```typescript
import OpenAI from "openai";
import Anthropic from "@anthropic-ai/sdk";

// OpenAI embeddings (phổ biến nhất)
const openai = new OpenAI();

async function embedText(text: string): Promise<number[]> {
  const response = await openai.embeddings.create({
    model: "text-embedding-3-small",  // 1536 dimensions, $0.02/1M tokens
    input: text,
  });
  return response.data[0].embedding;
}

// Batch embedding (hiệu quả hơn)
async function embedBatch(texts: string[]): Promise<number[][]> {
  const response = await openai.embeddings.create({
    model: "text-embedding-3-small",
    input: texts,
  });
  return response.data.map((d) => d.embedding);
}
```

**Embedding models so sánh:**

| Model | Dimensions | Cost | Quality |
|-------|-----------|------|---------|
| text-embedding-3-small | 1536 | $0.02/1M | Good |
| text-embedding-3-large | 3072 | $0.13/1M | Best (OpenAI) |
| voyage-3 (Anthropic) | 1024 | $0.06/1M | Best overall |
| all-MiniLM-L6-v2 | 384 | Free (local) | Good, fast |

---

### Step 4: Store vào Vector Database

```typescript
// Sau khi có chunks và embeddings, lưu vào vector DB
interface DocumentChunk {
  id: string;
  content: string;
  embedding: number[];
  metadata: {
    documentId: string;
    documentType: "policy" | "claim" | "underwriting";
    pageNumber?: number;
    section?: string;
    policyNumber?: string;
  };
}

// Indexing một document
async function indexDocument(filePath: string, metadata: any) {
  const text = await loadPDF(filePath);
  const chunks = await splitter.splitText(text);
  const embeddings = await embedBatch(chunks);

  const documents: DocumentChunk[] = chunks.map((content, i) => ({
    id: `${metadata.documentId}_chunk_${i}`,
    content,
    embedding: embeddings[i],
    metadata: { ...metadata, chunkIndex: i },
  }));

  await vectorDB.upsert(documents);
}
```

---

## 4. Retrieval Pipeline — chi tiết

### Dense Retrieval (Vector Search)

```typescript
async function retrieve(
  query: string,
  topK: number = 5,
  filter?: Record<string, any>
): Promise<DocumentChunk[]> {
  // 1. Embed the query
  const queryEmbedding = await embedText(query);

  // 2. Cosine similarity search
  const results = await vectorDB.query({
    vector: queryEmbedding,
    topK,
    filter,  // e.g., { policyNumber: "POL-001" } để filter specific policy
    includeMetadata: true,
  });

  return results.matches.map((m) => ({
    id: m.id,
    content: m.metadata.content,
    embedding: m.values,
    metadata: m.metadata,
    score: m.score,  // similarity score 0-1
  }));
}
```

### Hybrid Search (Dense + Sparse BM25) — tốt hơn

```typescript
// Dense: tốt cho semantic similarity ("car accident" ≈ "vehicle collision")
// Sparse (BM25/keyword): tốt cho exact match (policy numbers, names, dates)
// Hybrid: kết hợp cả hai

// Qdrant support hybrid search natively
const results = await qdrantClient.query("insurance_policies", {
  prefetch: [
    { query: queryEmbedding, using: "dense", limit: 20 },   // dense retrieval
    { query: sparseVector, using: "sparse", limit: 20 },     // BM25 keyword
  ],
  query: { fusion: "rrf" },  // Reciprocal Rank Fusion để merge results
  limit: 5,
});
```

### Re-ranking — optional nhưng cải thiện accuracy

```typescript
// Sau khi retrieve top-K, re-rank với cross-encoder (chậm hơn nhưng chính xác hơn)
// Cross-encoder đọc cả query và document cùng lúc — expensive nhưng accurate

import { Cohere } from "cohere-ai";
const cohere = new Cohere({ token: process.env.COHERE_API_KEY });

async function rerank(
  query: string,
  documents: DocumentChunk[],
  topN: number = 3
): Promise<DocumentChunk[]> {
  const response = await cohere.rerank({
    model: "rerank-english-v3.0",
    query,
    documents: documents.map((d) => d.content),
    topN,
  });

  return response.results.map((r) => documents[r.index]);
}
```

---

## 5. Generation với Retrieved Context

```typescript
async function ragQuery(
  userQuery: string,
  policyNumber?: string
): Promise<string> {
  // 1. Retrieve relevant chunks
  const chunks = await retrieve(
    userQuery,
    5,
    policyNumber ? { policyNumber } : undefined
  );

  // 2. Format context
  const context = chunks
    .map(
      (c, i) => `[Source ${i + 1}] ${c.metadata.section || ""}:\n${c.content}`
    )
    .join("\n\n---\n\n");

  // 3. Build prompt with context
  const systemPrompt = `
You are an insurance policy assistant.
Answer questions based ONLY on the policy context provided below.
If the answer is not in the context, say so clearly.
Always cite the source number [Source N] when referencing information.

POLICY CONTEXT:
${context}
`;

  // 4. Generate with LLM
  const response = await client.messages.create({
    model: "claude-sonnet-4-6",
    max_tokens: 1024,
    system: systemPrompt,
    messages: [{ role: "user", content: userQuery }],
  });

  return response.content[0].text;
}
```

---

## 6. Advanced RAG Patterns

### HyDE — Hypothetical Document Embedding

```typescript
// Vấn đề: Query ngắn không match tốt với document dài
// Giải pháp: Generate hypothetical answer, embed answer đó để search

async function hydeRetrieve(query: string): Promise<DocumentChunk[]> {
  // Step 1: Generate hypothetical document
  const hypothetical = await client.messages.create({
    model: "claude-haiku-4-5",  // dùng model rẻ cho step này
    max_tokens: 200,
    messages: [{
      role: "user",
      content: `Write a short paragraph that would answer: "${query}". 
                Focus on insurance terminology and procedures.`
    }]
  });

  const hypotheticalDoc = hypothetical.content[0].text;

  // Step 2: Embed hypothetical document (không phải query)
  return await retrieve(hypotheticalDoc, 5);
}
```

### Multi-Query Retrieval

```typescript
// Generate nhiều phiên bản của query → retrieve cho mỗi → merge & dedup

async function multiQueryRetrieve(query: string): Promise<DocumentChunk[]> {
  const variations = await client.messages.create({
    model: "claude-haiku-4-5",
    max_tokens: 200,
    messages: [{
      role: "user",
      content: `Generate 3 different ways to ask this insurance question:
                "${query}"
                Return as JSON array of strings.`
    }]
  });

  const queries: string[] = JSON.parse(variations.content[0].text);

  const allResults = await Promise.all(
    queries.map((q) => retrieve(q, 3))
  );

  // Dedup by chunk ID
  const seen = new Set<string>();
  return allResults.flat().filter((chunk) => {
    if (seen.has(chunk.id)) return false;
    seen.add(chunk.id);
    return true;
  });
}
```

---

## 7. Naive vs Advanced RAG

```
NAIVE RAG (đủ để start):
  Query → Embed → Top-K vector search → Inject → Generate

ADVANCED RAG (production):
  Query → Query rewriting → Multi-query retrieval → Hybrid search
       → Re-ranking → Filtered retrieval → Context compression
       → Inject → Generate → Hallucination check → Response

MODULAR RAG (state of art):
  Bất kỳ module nào có thể swap out — retriever, reranker, generator
  Cho phép A/B test từng component
```

---

## 8. RAG Evaluation Metrics

```typescript
// RAGAS — framework để evaluate RAG pipeline

// Key metrics:
// 1. Context Precision — retrieved chunks có relevant không?
//    = relevant_chunks_retrieved / total_chunks_retrieved

// 2. Context Recall — tất cả relevant info có được retrieve không?
//    = relevant_chunks_retrieved / total_relevant_chunks

// 3. Answer Faithfulness — answer có grounded trong context không?
//    = facts_in_answer_supported_by_context / total_facts_in_answer

// 4. Answer Relevance — answer có trả lời đúng câu hỏi không?
```

---

## Câu hỏi phỏng vấn

**Q: Chunk size nên là bao nhiêu?**
A: Không có một size phù hợp tất cả. Chunk nhỏ (256-512 char) = precision cao, context ít. Chunk lớn (1000-2000 char) = context nhiều, noise nhiều. Thường start với 512-1000 char + 10-20% overlap, sau đó evaluate bằng RAGAS và tune.

**Q: Khi nào retrieval fail?**
A: (1) Query và document dùng terminology khác nhau → dùng hybrid search hoặc query expansion. (2) Chunk boundary cắt giữa relevant information → tăng overlap, dùng semantic chunking. (3) Top-K quá nhỏ → tăng K nhưng cẩn thận context window và noise.

**Q: RAG vs Fine-tuning — khi nào dùng cái nào?**
A: RAG cho: data thay đổi thường xuyên, cần cite source, knowledge base lớn. Fine-tuning cho: style/format cần thay đổi, latency critical (ít prompt token hơn), task rất specific và stable. Thường kết hợp: fine-tune cho behavior, RAG cho knowledge.
