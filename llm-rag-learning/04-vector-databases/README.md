# Phase 04 — Vector Databases

> Vector DB = storage và search engine cho embeddings. Thay vì WHERE clause,
> bạn tìm kiếm bằng cosine similarity — "cho tôi những document gần nhất
> với query này trong không gian embedding."

---

## 1. Tại sao không dùng PostgreSQL thường?

```sql
-- Regular PostgreSQL: exact match, no semantic search
SELECT * FROM documents WHERE content LIKE '%insurance claim%';

-- pgvector: semantic similarity search
SELECT content, (embedding <=> query_embedding) AS distance
FROM documents
ORDER BY embedding <=> query_embedding
LIMIT 5;
-- tìm được "accident report" dù query là "car damage claim"
```

---

## 2. Các Vector DB phổ biến

| DB | Type | Pros | Cons | Best for |
|----|------|------|------|----------|
| **pgvector** | PostgreSQL extension | Bạn đã biết PG, ACID, SQL | Chậm hơn khi > 1M vectors | Startup, team có PG expertise |
| **Qdrant** | Dedicated, open source | Nhanh, hybrid search, free | Self-host overhead | Production, cần hybrid |
| **Pinecone** | Managed cloud | Không cần ops, scale tự động | Đắt, vendor lock-in | Enterprise, no-ops |
| **Weaviate** | Dedicated, open source | GraphQL, multi-modal | Phức tạp | Complex schema |
| **ChromaDB** | Local/embedded | Simple, free, zero setup | Không scale | Development, prototype |

---

## 3. pgvector — Start ở đây (bạn đã biết PostgreSQL)

### Setup

```sql
-- Enable extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Create table với vector column
CREATE TABLE document_chunks (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  document_id TEXT NOT NULL,
  content     TEXT NOT NULL,
  embedding   vector(1536),    -- 1536 cho text-embedding-3-small
  metadata    JSONB,
  created_at  TIMESTAMPTZ DEFAULT NOW()
);

-- Index cho vector search (HNSW — fastest query, slower build)
CREATE INDEX ON document_chunks
USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);
-- Hoặc IVFFlat — nhanh build hơn, chậm query hơn một chút
CREATE INDEX ON document_chunks
USING ivfflat (embedding vector_cosine_ops)
WITH (lists = 100);  -- lists ~ sqrt(total_rows)
```

### TypeScript với pgvector

```typescript
import { Pool } from "pg";

const pool = new Pool({ connectionString: process.env.DATABASE_URL });

// Upsert chunks
async function upsertChunk(chunk: {
  id: string;
  documentId: string;
  content: string;
  embedding: number[];
  metadata: Record<string, any>;
}) {
  await pool.query(
    `INSERT INTO document_chunks (id, document_id, content, embedding, metadata)
     VALUES ($1, $2, $3, $4::vector, $5)
     ON CONFLICT (id) DO UPDATE SET
       content = EXCLUDED.content,
       embedding = EXCLUDED.embedding,
       metadata = EXCLUDED.metadata`,
    [
      chunk.id,
      chunk.documentId,
      chunk.content,
      `[${chunk.embedding.join(",")}]`,  // pgvector format
      JSON.stringify(chunk.metadata),
    ]
  );
}

// Semantic search
async function search(
  queryEmbedding: number[],
  topK: number = 5,
  filter?: { documentType?: string; policyNumber?: string }
): Promise<any[]> {
  let whereClause = "";
  const params: any[] = [`[${queryEmbedding.join(",")}]`, topK];

  if (filter?.documentType) {
    params.push(filter.documentType);
    whereClause += ` AND metadata->>'documentType' = $${params.length}`;
  }
  if (filter?.policyNumber) {
    params.push(filter.policyNumber);
    whereClause += ` AND metadata->>'policyNumber' = $${params.length}`;
  }

  const result = await pool.query(
    `SELECT
       id, content, metadata,
       1 - (embedding <=> $1::vector) AS similarity
     FROM document_chunks
     WHERE 1=1 ${whereClause}
     ORDER BY embedding <=> $1::vector
     LIMIT $2`,
    params
  );

  return result.rows;
}
```

### TypeORM + pgvector (NestJS pattern — bạn đã dùng TypeORM)

```typescript
// Entity
import { Entity, Column, PrimaryGeneratedColumn } from "typeorm";

@Entity("document_chunks")
export class DocumentChunk {
  @PrimaryGeneratedColumn("uuid")
  id: string;

  @Column()
  documentId: string;

  @Column("text")
  content: string;

  @Column("simple-array")  // hoặc custom column type cho vector
  embedding: number[];

  @Column("jsonb")
  metadata: Record<string, any>;
}

// Custom vector type cho TypeORM
// Cần patch vì TypeORM chưa có native pgvector support
```

---

## 4. Qdrant — Production-grade open source

### Setup với Docker

```yaml
# docker-compose.yml
services:
  qdrant:
    image: qdrant/qdrant:latest
    ports:
      - "6333:6333"
    volumes:
      - ./qdrant_storage:/qdrant/storage
```

### TypeScript client

```typescript
import { QdrantClient } from "@qdrant/js-client-rest";

const qdrant = new QdrantClient({ url: "http://localhost:6333" });

// Create collection
await qdrant.createCollection("insurance_docs", {
  vectors: {
    dense: {
      size: 1536,
      distance: "Cosine",
    },
    sparse: {                    // cho hybrid search
      index: { on_disk: false },
    },
  },
});

// Upsert points
await qdrant.upsert("insurance_docs", {
  points: chunks.map((chunk) => ({
    id: chunk.id,
    vector: { dense: chunk.embedding },
    payload: {
      content: chunk.content,
      documentId: chunk.documentId,
      documentType: chunk.metadata.documentType,
      policyNumber: chunk.metadata.policyNumber,
    },
  })),
});

// Search với filter
const results = await qdrant.query("insurance_docs", {
  query: queryEmbedding,
  using: "dense",
  filter: {
    must: [
      { key: "documentType", match: { value: "policy" } },
      { key: "policyNumber", match: { value: "POL-2024-001" } },
    ],
  },
  limit: 5,
  with_payload: true,
});

// Hybrid search (dense + sparse)
const hybridResults = await qdrant.query("insurance_docs", {
  prefetch: [
    { query: denseEmbedding, using: "dense", limit: 20 },
    { query: { indices: sparseIndices, values: sparseValues }, using: "sparse", limit: 20 },
  ],
  query: { fusion: "rrf" },  // Reciprocal Rank Fusion
  limit: 5,
});
```

---

## 5. ChromaDB — Development / Prototype

```typescript
import { ChromaClient, OpenAIEmbeddingFunction } from "chromadb";

const chroma = new ChromaClient({ path: "http://localhost:8000" });

const embeddingFunction = new OpenAIEmbeddingFunction({
  openai_api_key: process.env.OPENAI_API_KEY,
  openai_model: "text-embedding-3-small",
});

// Get or create collection — handles embedding automatically
const collection = await chroma.getOrCreateCollection({
  name: "insurance_docs",
  embeddingFunction,
});

// Add documents (embedding happens automatically)
await collection.add({
  ids: ["chunk_1", "chunk_2"],
  documents: ["Policy covers accidental damage...", "Claims must be filed within 30 days..."],
  metadatas: [
    { documentType: "policy", section: "coverage" },
    { documentType: "policy", section: "claims_process" },
  ],
});

// Query
const results = await collection.query({
  queryTexts: ["What does my policy cover?"],
  nResults: 5,
  where: { documentType: "policy" },  // metadata filter
});
```

---

## 6. NestJS Service Pattern — Vector DB abstraction

```typescript
// Abstraction layer — swap DB dễ dàng
export interface VectorStore {
  upsert(chunks: DocumentChunk[]): Promise<void>;
  search(
    query: number[],
    topK: number,
    filter?: Record<string, any>
  ): Promise<SearchResult[]>;
  delete(ids: string[]): Promise<void>;
}

// Implementation với pgvector
@Injectable()
export class PgVectorStore implements VectorStore {
  constructor(@InjectDataSource() private dataSource: DataSource) {}

  async upsert(chunks: DocumentChunk[]): Promise<void> {
    // ... pgvector implementation
  }

  async search(query: number[], topK: number, filter?: Record<string, any>) {
    // ... pgvector search
  }
}

// Module
@Module({
  providers: [
    {
      provide: "VECTOR_STORE",
      useClass: PgVectorStore,  // swap to QdrantVectorStore easily
    },
  ],
  exports: ["VECTOR_STORE"],
})
export class VectorStoreModule {}

// Service dùng abstraction
@Injectable()
export class RAGService {
  constructor(@Inject("VECTOR_STORE") private vectorStore: VectorStore) {}

  async query(userQuery: string): Promise<string> {
    const embedding = await this.embedQuery(userQuery);
    const chunks = await this.vectorStore.search(embedding, 5);
    return this.generateAnswer(userQuery, chunks);
  }
}
```

---

## 7. Metadata Filtering — quan trọng cho Insurance

```typescript
// Insurance system cần filter rất cụ thể:
// - Chỉ search trong policy của khách hàng này
// - Chỉ search claim documents của loại này
// - Chỉ search documents updated trong 1 năm gần đây

interface InsuranceFilter {
  policyNumber?: string;
  customerId?: string;
  documentType?: "policy" | "claim" | "underwriting" | "fwa_report";
  insuranceType?: "life" | "non_life" | "health" | "auto";
  dateRange?: { from: Date; to: Date };
}

async function insuranceRAGSearch(
  query: string,
  filter: InsuranceFilter
): Promise<string> {
  const embedding = await embedText(query);

  // Build filter dynamically
  const qdrantFilter = buildQdrantFilter(filter);

  const results = await qdrant.query("insurance_docs", {
    query: embedding,
    filter: qdrantFilter,
    limit: 5,
  });

  // Generate answer
  const context = results.map((r) => r.payload.content).join("\n\n");
  return generateAnswer(query, context);
}
```

---

## Câu hỏi phỏng vấn

**Q: Chọn pgvector hay Qdrant?**
A: pgvector nếu team đã có PostgreSQL và vector data < 1M rows — ít infrastructure hơn, ACID transactions, familiar SQL. Qdrant nếu cần hybrid search, scale lớn hơn, hoặc performance critical. Với startup InsurTech, pgvector đủ để start, migrate sang Qdrant khi cần.

**Q: Cosine similarity vs dot product vs Euclidean distance?**
A: Cosine similarity (vector <=> query) = phổ biến nhất cho text embedding — measure góc giữa 2 vector, không phụ thuộc magnitude. Dùng cosine cho normalized embeddings (OpenAI, Claude embeddings đều normalized). Euclidean kém hơn cho high-dimensional data.

**Q: Làm sao update knowledge base khi document thay đổi?**
A: Delete old chunks theo documentId, re-embed và re-insert. Với pgvector: `DELETE FROM document_chunks WHERE document_id = $1` trước khi insert mới. Cần track documentId và version để know khi nào update.
