# Phase 11 — Advanced Retrieval

> Naive RAG (embed → cosine search → inject) đủ để start, nhưng production system
> cần nhiều hơn: GraphRAG, context compression, self-RAG, corrective RAG, và
> các kỹ thuật reranking nâng cao. Phase này cover những gì naive RAG không làm được.

---

## 1. Giới hạn của Naive RAG

```
Vấn đề 1: Multi-hop reasoning
  Q: "Khách hàng mua policy năm nào, và coverage có áp dụng cho sự kiện xảy ra
     trước khi có policy không?"
  → Naive RAG retrieve 1 document về event, 1 về policy terms
  → Nhưng không "kết nối" fact: policy date ↔ event date
  → Cần GraphRAG hoặc multi-step retrieval

Vấn đề 2: Context too long → performance drop
  → Inject 5 chunks × 1000 chars = 5000 chars context
  → LLM attention "loãng" khi context dài
  → Cần context compression

Vấn đề 3: Retrieved chunks có thể không relevant
  → Vector search trả về "gần nhất" không phải "relevant nhất"
  → Cần reranking

Vấn đề 4: Query ambiguous
  → "coverage" → bảo hiểm gì? của ai? khi nào?
  → Cần query understanding trước khi retrieve
```

---

## 2. Advanced Reranking — Sau Vector Search

### Cohere Rerank (managed, production-ready)

```typescript
import { CohereClient } from "cohere-ai";

const cohere = new CohereClient({ token: process.env.COHERE_API_KEY });

async function rerankWithCohere(
  query: string,
  candidates: SearchResult[],
  topN: number = 3
): Promise<SearchResult[]> {
  const response = await cohere.rerank({
    model: "rerank-english-v3.0",
    query,
    documents: candidates.map((c) => c.content),
    topN,
    returnDocuments: false,  // chỉ trả về indices
  });

  return response.results.map((r) => ({
    ...candidates[r.index],
    rerankScore: r.relevanceScore,
  }));
}

// Pipeline với rerank:
async function retrieveWithRerank(query: string): Promise<SearchResult[]> {
  // 1. Over-retrieve (lấy nhiều để rerank có nhiều lựa chọn)
  const candidates = await vectorStore.search(queryEmbedding, 20);  // top-20

  // 2. Rerank xuống top-3
  const reranked = await rerankWithCohere(query, candidates, 3);

  return reranked;
}
```

### Cross-Encoder (self-hosted)

```typescript
// Cross-encoder đọc query VÀ document cùng lúc — chính xác hơn bi-encoder
// nhưng chậm hơn O(N) so với bi-encoder O(1) sau indexing

// Dùng với Hugging Face Inference API
async function crossEncoderRerank(
  query: string,
  documents: string[]
): Promise<number[]> {
  const response = await fetch(
    "https://api-inference.huggingface.co/models/cross-encoder/ms-marco-MiniLM-L-6-v2",
    {
      method: "POST",
      headers: { Authorization: `Bearer ${process.env.HF_TOKEN}` },
      body: JSON.stringify({
        inputs: documents.map((doc) => ({ text: query, text_pair: doc })),
      }),
    }
  );

  const scores: number[] = await response.json();
  return scores;
}
```

---

## 3. Context Compression — Giảm noise, giữ relevant info

### LLM-based Compression

```typescript
// Thay vì inject toàn bộ chunk (có nhiều phần không relevant),
// compress mỗi chunk xuống chỉ giữ phần relevant với query

async function compressChunk(
  query: string,
  chunk: string
): Promise<string | null> {
  const response = await client.messages.create({
    model: "claude-haiku-4-5",  // model rẻ cho compression
    max_tokens: 300,
    messages: [{
      role: "user",
      content: `Extract ONLY the information relevant to this question from the passage.
If no relevant information exists, return "NOT_RELEVANT".

Question: ${query}

Passage:
${chunk}

Relevant extract:`,
    }],
  });

  const result = response.content[0].text.trim();
  return result === "NOT_RELEVANT" ? null : result;
}

async function retrieveAndCompress(query: string): Promise<string> {
  // 1. Retrieve more chunks (compensate for some being filtered)
  const chunks = await vectorStore.search(queryEmbedding, 10);

  // 2. Compress in parallel
  const compressed = await Promise.all(
    chunks.map((c) => compressChunk(query, c.content))
  );

  // 3. Filter out null (not relevant) and join
  return compressed.filter(Boolean).slice(0, 5).join("\n\n");
}
```

### LLMLingua — Token-level Compression

```typescript
// LLMLingua: compress prompt bằng cách remove unimportant tokens
// Giảm 3-5x số tokens, giữ 95%+ performance
// Source: microsoft/LLMLingua (Python library)

// TypeScript: call Python service qua HTTP
const response = await fetch("http://llmlingua-service:5000/compress", {
  method: "POST",
  body: JSON.stringify({
    context: largeContext,
    question: query,
    target_token: 500,           // target token count sau compression
    condition_in_question: true,
    reorder_context: "sort",
  }),
});

const { compressed_prompt } = await response.json();
// compressed_prompt có ~500 tokens thay vì 2000+ tokens
```

---

## 4. GraphRAG — Knowledge Graph + RAG

### Tại sao cần Graph?

```
Flat RAG: documents → chunks → search by similarity
→ Mỗi chunk độc lập, không biết relationship

GraphRAG: documents → entities + relationships → graph
→ "Policy POL-001 has Coverage auto → Coverage auto excludes racing → Racing happened at event X"
→ Multi-hop reasoning qua graph
```

### Concept và Implementation

```typescript
// GraphRAG pipeline:
// 1. Entity extraction từ documents
// 2. Relationship extraction
// 3. Build knowledge graph
// 4. Query graph + vector search combined

// Step 1: Extract entities và relationships
interface Entity {
  id: string;
  type: "Policy" | "Claim" | "Customer" | "Coverage" | "Exclusion" | "Event";
  name: string;
  properties: Record<string, any>;
}

interface Relationship {
  fromId: string;
  toId: string;
  type: string;
  properties?: Record<string, any>;
}

async function extractEntitiesAndRelationships(text: string) {
  const response = await client.messages.create({
    model: "claude-sonnet-4-6",
    max_tokens: 2048,
    messages: [{
      role: "user",
      content: `Extract entities and relationships from this insurance document.

Return JSON:
{
  "entities": [
    { "id": "unique_id", "type": "Policy|Claim|Customer|Coverage|Exclusion|Event", "name": "...", "properties": {} }
  ],
  "relationships": [
    { "fromId": "...", "toId": "...", "type": "HAS_COVERAGE|FILED_CLAIM|EXCLUDES|OCCURRED_DURING|...", "properties": {} }
  ]
}

Document:
${text}`,
    }],
  });

  return JSON.parse(response.content[0].text);
}

// Step 2: Store in graph database (Neo4j) hoặc in-memory
// Neo4j với TypeScript:
import neo4j from "neo4j-driver";

const driver = neo4j.driver(
  process.env.NEO4J_URI!,
  neo4j.auth.basic(process.env.NEO4J_USER!, process.env.NEO4J_PASSWORD!)
);

async function storeGraph(entities: Entity[], relationships: Relationship[]) {
  const session = driver.session();

  // Create entities
  for (const entity of entities) {
    await session.run(
      `MERGE (n:${entity.type} { id: $id })
       SET n += $properties, n.name = $name`,
      { id: entity.id, name: entity.name, properties: entity.properties }
    );
  }

  // Create relationships
  for (const rel of relationships) {
    await session.run(
      `MATCH (a { id: $fromId }), (b { id: $toId })
       MERGE (a)-[r:${rel.type}]->(b)
       SET r += $properties`,
      { fromId: rel.fromId, toId: rel.toId, properties: rel.properties ?? {} }
    );
  }

  await session.close();
}

// Step 3: GraphRAG query
async function graphRAGQuery(query: string, customerId: string): Promise<string> {
  const session = driver.session();

  // Traverse graph to find relevant info
  // Example: find all coverages and exclusions for customer's policies
  const graphResult = await session.run(
    `MATCH (c:Customer { id: $customerId })-[:HAS_POLICY]->(p:Policy)
     MATCH (p)-[:HAS_COVERAGE]->(cov:Coverage)
     OPTIONAL MATCH (cov)-[:EXCLUDES]->(exc:Exclusion)
     RETURN p.name, cov.type, cov.limit, cov.deductible, collect(exc.description) as exclusions`,
    { customerId }
  );

  const graphContext = graphResult.records
    .map((r) => `Policy: ${r.get("p.name")}, Coverage: ${r.get("cov.type")}, Limit: ${r.get("cov.limit")}, Exclusions: ${r.get("exclusions").join(", ")}`)
    .join("\n");

  // Combine with vector search
  const vectorChunks = await vectorStore.search(await embedText(query), 3, { customerId });
  const vectorContext = vectorChunks.map((c) => c.content).join("\n\n");

  // Generate with combined context
  return llm.complete(
    [{ role: "user", content: query }],
    {
      system: `Answer using these sources:

GRAPH DATA (structured relationships):
${graphContext}

DOCUMENT CONTEXT (detailed text):
${vectorContext}

Synthesize both sources for a complete answer.`,
      temperature: 0,
    }
  );
}
```

### Microsoft GraphRAG (Open Source)

```bash
# Microsoft đã open-source GraphRAG pipeline
pip install graphrag

# Config và run:
graphrag init --root ./insurance-docs
graphrag index --root ./insurance-docs
graphrag query --root ./insurance-docs --method global "What are common fraud patterns?"
```

---

## 5. Self-RAG — Model tự quyết định khi nào retrieve

```typescript
// Thay vì luôn retrieve, model tự quyết định:
// 1. Có cần retrieve không? (có thể trả lời từ training knowledge)
// 2. Retrieved document có relevant không?
// 3. Answer có supported bởi context không?

// Implementation đơn giản với reflection tokens
async function selfRAGQuery(query: string): Promise<string> {
  // Step 1: Ask model nếu cần retrieval
  const needsRetrieval = await client.messages.create({
    model: "claude-haiku-4-5",
    max_tokens: 50,
    messages: [{
      role: "user",
      content: `Does answering this question require looking up specific insurance policy documents, claim details, or customer-specific information?
Question: ${query}
Answer with YES or NO and brief reason.`
    }]
  });

  const shouldRetrieve = needsRetrieval.content[0].text.trim().startsWith("YES");

  if (!shouldRetrieve) {
    // Answer from training knowledge (general insurance question)
    return client.messages.create({
      model: "claude-sonnet-4-6",
      max_tokens: 1024,
      messages: [{ role: "user", content: query }],
    }).then(r => r.content[0].text);
  }

  // Step 2: Retrieve
  const chunks = await retrieve(query, 5);

  // Step 3: Check relevance của mỗi chunk
  const relevantChunks = await Promise.all(
    chunks.map(async (chunk) => {
      const isRelevant = await client.messages.create({
        model: "claude-haiku-4-5",
        max_tokens: 20,
        messages: [{
          role: "user",
          content: `Is this passage relevant to answering: "${query}"?\n\nPassage: ${chunk.content}\n\nAnswer YES or NO.`
        }]
      });
      return isRelevant.content[0].text.trim().startsWith("YES") ? chunk : null;
    })
  );

  const filteredChunks = relevantChunks.filter(Boolean) as SearchResult[];

  if (filteredChunks.length === 0) {
    return "I could not find relevant information in the policy documents to answer this question.";
  }

  // Step 4: Generate with relevant context only
  const context = filteredChunks.map(c => c.content).join("\n\n");

  return client.messages.create({
    model: "claude-sonnet-4-6",
    max_tokens: 1024,
    system: `Answer based ONLY on the provided context:\n${context}`,
    messages: [{ role: "user", content: query }],
  }).then(r => r.content[0].text);
}
```

---

## 6. Corrective RAG (CRAG) — Tự fix khi retrieve sai

```typescript
// CRAG: Evaluate retrieved docs quality, correct nếu quality thấp

type RetrievalQuality = "CORRECT" | "INCORRECT" | "AMBIGUOUS";

async function correctiveRAG(query: string): Promise<string> {
  // 1. Retrieve
  const chunks = await retrieve(query, 5);

  // 2. Evaluate retrieval quality
  const evaluation = await client.messages.create({
    model: "claude-haiku-4-5",
    max_tokens: 100,
    messages: [{
      role: "user",
      content: `Evaluate if these retrieved passages are relevant to answer the question.
Question: ${query}
Passages: ${chunks.map(c => c.content.slice(0, 200)).join("\n---\n")}

Return JSON: { "quality": "CORRECT|INCORRECT|AMBIGUOUS", "reason": "..." }`
    }]
  });

  const { quality }: { quality: RetrievalQuality } = JSON.parse(
    evaluation.content[0].text
  );

  if (quality === "CORRECT") {
    // Proceed with retrieved context
    return generateAnswer(query, chunks.map(c => c.content).join("\n\n"));

  } else if (quality === "INCORRECT") {
    // Fall back to web search hoặc broader knowledge base
    const webResults = await webSearch(query);  // implement web search
    return generateAnswer(query, webResults);

  } else {
    // AMBIGUOUS: combine retrieved + web search
    const webResults = await webSearch(query);
    const combinedContext = [
      ...chunks.map(c => c.content),
      webResults,
    ].join("\n\n");
    return generateAnswer(query, combinedContext);
  }
}
```

---

## 7. Parent Document Retrieval

```typescript
// Vấn đề: chunk nhỏ → precise retrieval nhưng mất context
// Giải pháp: embed small chunks, nhưng retrieve parent chunk lớn hơn

interface DocumentNode {
  id: string;
  content: string;
  parentId?: string;
  children: string[];
  level: "document" | "section" | "paragraph" | "sentence";
}

// Indexing: embed small chunks, store reference to parent
async function indexWithParentRetrieval(document: string, docId: string) {
  // Split thành sections (large) và sentences (small)
  const sections = splitIntoSections(document);      // ~2000 chars each
  const sentences = splitIntoSentences(document);    // ~200 chars each

  // Only embed sentences (small) for retrieval precision
  const sentenceEmbeddings = await embedBatch(sentences.map(s => s.content));

  await vectorStore.upsert(sentences.map((s, i) => ({
    id: `${docId}_sent_${i}`,
    content: s.content,
    embedding: sentenceEmbeddings[i],
    metadata: {
      docId,
      parentSectionId: s.parentSectionId,  // reference to parent
    },
  })));

  // Store sections separately (not indexed for search)
  await sectionStore.save(sections);
}

// Retrieval: search small, return large
async function retrieveWithParent(query: string): Promise<string[]> {
  // 1. Search small chunks (precise)
  const smallChunks = await vectorStore.search(queryEmbedding, 5);

  // 2. Get parent sections for each found chunk
  const parentIds = [...new Set(smallChunks.map(c => c.metadata.parentSectionId))];
  const parentSections = await sectionStore.findByIds(parentIds);

  // 3. Return full sections (much richer context)
  return parentSections.map(s => s.content);
}
```

---

## 8. So sánh các Advanced Retrieval Strategies

| Strategy | Khi nào dùng | Overhead | Improvement |
|----------|-------------|----------|-------------|
| **Reranking (Cohere)** | Luôn dùng nếu budget cho phép | Low (~100ms) | High |
| **Context Compression** | Context window tight, chunks noisy | Medium | Medium-High |
| **Parent Retrieval** | Chunks quá nhỏ, mất context | Low | Medium |
| **HyDE** | Short queries không match long docs | +1 LLM call | Medium |
| **Multi-Query** | Ambiguous queries | +1 LLM call × N | Medium |
| **Self-RAG** | Mixed questions (some need retrieval) | +1-3 LLM calls | High |
| **CRAG** | Low-quality knowledge base | +1-2 LLM calls | Medium |
| **GraphRAG** | Multi-hop reasoning, relationships | High setup | Very High |

**Recommended stack cho InsurTech:**
```
1. Hybrid Search (dense + sparse) — baseline
2. Cohere Reranking — always add this
3. Parent Retrieval — when chunks are small
4. Context Compression — when context > 4000 tokens
5. GraphRAG — for complex policy-claim-customer relationships
```

---

## Câu hỏi phỏng vấn

**Q: Naive RAG cho kết quả kém — bạn sẽ debug và improve như thế nào?**
A: Systematic approach: (1) Log retrieval results — xem chunks nào đang được retrieve, có relevant không. (2) Nếu chunks không relevant → improve chunking (semantic chunking), thêm metadata filtering, thử hybrid search. (3) Nếu chunks relevant nhưng answer sai → add reranking (Cohere), context compression, improve system prompt. (4) Nếu query phức tạp → thêm query rewriting, multi-query retrieval, hoặc self-RAG. Đo bằng RAGAS metrics ở mỗi bước.

**Q: GraphRAG expensive to setup — có đáng không?**
A: Depends on use case. Cho InsurTech: nếu queries cần kết nối nhiều entities (policy + claims + coverage + exclusions), GraphRAG rất valuable. Nếu queries đơn giản (lookup single document), naive RAG + reranking đủ. Start với naive RAG + reranking, add GraphRAG chỉ khi evaluation shows multi-hop failures.
