# Phase 07 — Production Patterns

> Difference giữa demo và production AI system: evaluation, observability,
> cost control, guardrails, và reliability. Phase này cover những gì cần có
> trước khi ship GenAI feature cho real users.

---

## 1. Evaluation — Làm sao biết RAG của mình hoạt động tốt?

### LLM-as-Judge pattern

```typescript
// Dùng LLM để evaluate output của LLM khác
interface EvalResult {
  score: number;       // 0-1
  reasoning: string;
  passed: boolean;
}

async function evaluateRAGAnswer(
  question: string,
  context: string,
  answer: string
): Promise<{ faithfulness: EvalResult; relevance: EvalResult }> {

  const faithfulnessPrompt = `
You are evaluating if an AI answer is faithful to the provided context.
Faithfulness = the answer only contains facts supported by the context.

Context: ${context}
Answer: ${answer}

Rate faithfulness 0-1 and explain. Return JSON: { score: number, reasoning: string }
`;

  const relevancePrompt = `
You are evaluating if an AI answer actually answers the question.
Relevance = the answer addresses what was asked.

Question: ${question}
Answer: ${answer}

Rate relevance 0-1 and explain. Return JSON: { score: number, reasoning: string }
`;

  const [faithfulness, relevance] = await Promise.all([
    llm.complete([{ role: "user", content: faithfulnessPrompt }], { temperature: 0 }),
    llm.complete([{ role: "user", content: relevancePrompt }], { temperature: 0 }),
  ]);

  const faithfulnessResult = JSON.parse(faithfulness);
  const relevanceResult = JSON.parse(relevance);

  return {
    faithfulness: { ...faithfulnessResult, passed: faithfulnessResult.score >= 0.8 },
    relevance: { ...relevanceResult, passed: relevanceResult.score >= 0.7 },
  };
}
```

### Golden Dataset — Regression Testing

```typescript
// Create test cases với known good answers
interface TestCase {
  id: string;
  question: string;
  policyNumber?: string;
  expectedKeywords: string[];  // answer phải contain những từ này
  expectedFaithfulness: number;
  expectedRelevance: number;
}

const GOLDEN_DATASET: TestCase[] = [
  {
    id: "TC001",
    question: "What is the deductible for auto claims?",
    policyNumber: "POL-TEST-001",
    expectedKeywords: ["deductible", "$500", "collision"],
    expectedFaithfulness: 0.9,
    expectedRelevance: 0.9,
  },
  // ... more test cases
];

async function runEvalSuite(): Promise<EvalReport> {
  const results = await Promise.all(
    GOLDEN_DATASET.map(async (tc) => {
      const answer = await ragService.query(tc.question, { filter: { policyNumber: tc.policyNumber } });
      const eval_ = await evaluateRAGAnswer(tc.question, answer.context, answer.text);

      return {
        testId: tc.id,
        passed:
          eval_.faithfulness.passed &&
          eval_.relevance.passed &&
          tc.expectedKeywords.every((kw) => answer.text.toLowerCase().includes(kw.toLowerCase())),
        ...eval_,
      };
    })
  );

  return {
    totalTests: results.length,
    passed: results.filter((r) => r.passed).length,
    failed: results.filter((r) => !r.passed),
    avgFaithfulness: results.reduce((s, r) => s + r.faithfulness.score, 0) / results.length,
    avgRelevance: results.reduce((s, r) => s + r.relevance.score, 0) / results.length,
  };
}
```

---

## 2. Observability — Track mọi LLM call

### LLM Call Logging

```typescript
// NestJS interceptor để log tất cả LLM calls
@Injectable()
export class LLMObservabilityInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const start = Date.now();

    return next.handle().pipe(
      tap((response) => {
        const duration = Date.now() - start;
        this.logLLMCall({
          timestamp: new Date(),
          duration,
          model: response.model,
          inputTokens: response.usage?.input_tokens,
          outputTokens: response.usage?.output_tokens,
          cost: this.calculateCost(response),
          endpoint: context.switchToHttp().getRequest().url,
          requestId: context.switchToHttp().getRequest().id,
        });
      })
    );
  }
}

// Wrapper service với built-in logging
@Injectable()
export class ObservableLLMService {
  private readonly logger = new Logger(ObservableLLMService.name);

  async complete(messages: any[], options: any = {}): Promise<LLMResponse> {
    const traceId = uuid();
    const start = Date.now();

    this.logger.log({ event: "llm_call_start", traceId, model: "claude-sonnet-4-6" });

    try {
      const response = await this.client.messages.create({
        model: "claude-sonnet-4-6",
        ...options,
        messages,
      });

      const duration = Date.now() - start;
      const cost = this.calculateCost(response.usage);

      this.logger.log({
        event: "llm_call_complete",
        traceId,
        duration,
        inputTokens: response.usage.input_tokens,
        outputTokens: response.usage.output_tokens,
        costUSD: cost,
      });

      // OpenTelemetry span — bạn đã biết OpenTelemetry từ Herond
      this.tracer.startActiveSpan("llm.complete", (span) => {
        span.setAttributes({
          "llm.model": "claude-sonnet-4-6",
          "llm.input_tokens": response.usage.input_tokens,
          "llm.output_tokens": response.usage.output_tokens,
          "llm.cost_usd": cost,
          "llm.duration_ms": duration,
        });
        span.end();
      });

      return response;
    } catch (err) {
      this.logger.error({ event: "llm_call_error", traceId, error: err.message });
      throw err;
    }
  }

  private calculateCost(usage: { input_tokens: number; output_tokens: number }): number {
    // Claude Sonnet 4.6 pricing
    const inputCost = (usage.input_tokens / 1_000_000) * 3;
    const outputCost = (usage.output_tokens / 1_000_000) * 15;
    return inputCost + outputCost;
  }
}
```

---

## 3. Caching — Giảm cost và latency

### Semantic Caching

```typescript
// Cache LLM responses dựa trên semantic similarity của query
// Nếu query mới "gần giống" query cũ → return cached response

@Injectable()
export class SemanticCacheService {
  private SIMILARITY_THRESHOLD = 0.95;  // rất gần mới dùng cache

  constructor(
    private embedding: EmbeddingService,
    @InjectRepository(CacheEntry) private cacheRepo: Repository<CacheEntry>,
    private redis: Redis  // Redis TTL cho short-term cache
  ) {}

  async get(query: string): Promise<string | null> {
    // 1. Check exact match in Redis (fast)
    const exactKey = `llm:${createHash("md5").update(query).digest("hex")}`;
    const exact = await this.redis.get(exactKey);
    if (exact) return exact;

    // 2. Semantic similarity check (slower, more powerful)
    const queryEmbedding = await this.embedding.embed(query);
    const similar = await this.cacheRepo
      .createQueryBuilder("c")
      .where(
        `1 - (c.query_embedding <=> :embedding::vector) > :threshold`,
        { embedding: `[${queryEmbedding.join(",")}]`, threshold: this.SIMILARITY_THRESHOLD }
      )
      .orderBy(`c.query_embedding <=> :embedding::vector`)
      .getOne();

    return similar?.response ?? null;
  }

  async set(query: string, response: string): Promise<void> {
    const queryEmbedding = await this.embedding.embed(query);

    // Save to DB for semantic search
    await this.cacheRepo.save({
      query,
      response,
      queryEmbedding: `[${queryEmbedding.join(",")}]`,
    });

    // Also save to Redis for 1 hour exact match
    const exactKey = `llm:${createHash("md5").update(query).digest("hex")}`;
    await this.redis.setex(exactKey, 3600, response);
  }
}
```

### Prompt Caching (Anthropic feature)

```typescript
// Anthropic cache_control — cache system prompt + large context
// Giảm cost 90% cho repeated long prompts
const response = await client.messages.create({
  model: "claude-opus-4-8",
  max_tokens: 1024,
  system: [
    {
      type: "text",
      text: LARGE_INSURANCE_POLICY_DOCUMENT,  // 50k tokens
      cache_control: { type: "ephemeral" },   // cache này — chỉ tính tiền lần đầu
    },
  ],
  messages: [{ role: "user", content: userQuestion }],
});
// Lần đầu: tính tiền 50k token. Lần sau (trong 5 phút): 90% cheaper
```

---

## 4. Guardrails — Bảo vệ output

```typescript
// Output validation và safety checks
@Injectable()
export class GuardrailService {

  // 1. Schema validation (bạn đã biết Zod từ NestJS)
  async validateStructuredOutput<T>(
    output: string,
    schema: z.ZodSchema<T>
  ): Promise<T> {
    try {
      return schema.parse(JSON.parse(output));
    } catch {
      throw new InvalidLLMOutputError("LLM output failed schema validation");
    }
  }

  // 2. Content safety check
  async checkContentSafety(text: string): Promise<boolean> {
    const response = await this.client.messages.create({
      model: "claude-haiku-4-5",  // fast, cheap model for safety check
      max_tokens: 10,
      messages: [{
        role: "user",
        content: `Does this text contain harmful, discriminatory, or inappropriate content for an insurance context?
Text: "${text}"
Answer with only YES or NO.`
      }]
    });

    return response.content[0].text.trim().toUpperCase() === "NO";
  }

  // 3. Hallucination detection — claim specific facts are in context
  async detectHallucination(
    answer: string,
    sourceContext: string
  ): Promise<{ isHallucinated: boolean; confidence: number }> {
    const result = await this.llm.complete(
      [{
        role: "user",
        content: `Does the answer contain any facts NOT supported by the context?
Context: ${sourceContext}
Answer: ${answer}
Return JSON: { isHallucinated: boolean, confidence: number, unsupportedFacts: string[] }`
      }],
      { temperature: 0 }
    );

    return JSON.parse(result);
  }

  // 4. PII detection — critical cho insurance (GDPR, data privacy)
  async detectPII(text: string): Promise<{ hasPII: boolean; types: string[] }> {
    // Regex patterns cho quick check trước khi LLM
    const patterns = {
      email: /\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b/,
      phone: /\b\d{10,11}\b|\b\d{3}[-.\s]\d{3}[-.\s]\d{4}\b/,
      ssn: /\b\d{3}-?\d{2}-?\d{4}\b/,
      creditCard: /\b\d{4}[- ]?\d{4}[- ]?\d{4}[- ]?\d{4}\b/,
    };

    const detected = Object.entries(patterns)
      .filter(([, regex]) => regex.test(text))
      .map(([type]) => type);

    return {
      hasPII: detected.length > 0,
      types: detected,
    };
  }
}
```

---

## 5. Rate Limiting và Retry

```typescript
// Exponential backoff retry với p-retry
import pRetry from "p-retry";

@Injectable()
export class RobustLLMService {
  async complete(messages: any[], options: any = {}): Promise<string> {
    return pRetry(
      async () => {
        try {
          const response = await this.client.messages.create({
            model: "claude-sonnet-4-6",
            max_tokens: 1024,
            messages,
            ...options,
          });
          return response.content[0].text;
        } catch (err) {
          // Rate limit — retry với backoff
          if (err.status === 429) throw err;
          // Other errors — don't retry
          throw new pRetry.AbortError(err);
        }
      },
      {
        retries: 3,
        minTimeout: 1000,
        factor: 2,       // 1s, 2s, 4s
        onFailedAttempt: (err) => {
          this.logger.warn(`LLM call failed, attempt ${err.attemptNumber}: ${err.message}`);
        },
      }
    );
  }
}
```

---

## 6. Cost Control

```typescript
// Budget tracker — không cho vượt daily cost limit
@Injectable()
export class CostBudgetService {
  private readonly DAILY_LIMIT_USD = 50;

  async checkBudget(): Promise<void> {
    const today = new Date().toISOString().split("T")[0];
    const spent = await this.redis.get(`llm:cost:${today}`);

    if (parseFloat(spent ?? "0") >= this.DAILY_LIMIT_USD) {
      throw new BudgetExceededError(
        `Daily LLM budget of $${this.DAILY_LIMIT_USD} exceeded`
      );
    }
  }

  async recordCost(costUSD: number): Promise<void> {
    const today = new Date().toISOString().split("T")[0];
    await this.redis.incrbyfloat(`llm:cost:${today}`, costUSD);
    await this.redis.expire(`llm:cost:${today}`, 86400 * 2);  // 2 day TTL
  }
}

// Model routing — dùng model rẻ hơn khi có thể
function selectModel(task: LLMTask): string {
  switch (task.complexity) {
    case "simple":      return "claude-haiku-4-5";   // $0.80/1M input
    case "moderate":    return "claude-sonnet-4-6";  // $3/1M input
    case "complex":     return "claude-opus-4-8";    // $15/1M input
    default:            return "claude-sonnet-4-6";
  }
}
```

---

## 7. Checklist trước khi ship GenAI feature

```
□ Evaluation
  □ Golden dataset created với 20+ test cases
  □ Faithfulness score > 0.85 on test set
  □ Relevance score > 0.80 on test set
  □ Regression test in CI/CD pipeline

□ Observability
  □ Mọi LLM call được logged (tokens, cost, duration, requestId)
  □ Dashboard hiển thị daily cost, p95 latency, error rate
  □ Alert khi cost vượt threshold

□ Safety
  □ Input sanitization (prompt injection defense)
  □ Output schema validation với retry
  □ PII detection trước khi log
  □ Content safety check cho user-facing output

□ Reliability
  □ Retry với exponential backoff
  □ Graceful degradation khi LLM down
  □ Timeout configuration phù hợp
  □ Streaming cho response > 2 seconds

□ Cost
  □ Semantic caching implemented
  □ Model routing (haiku cho simple tasks)
  □ Daily budget alert configured
  □ Token counting để estimate cost trước khi call
```

---

## Câu hỏi phỏng vấn

**Q: Làm sao monitor AI feature in production?**
A: Log mọi LLM call với traceId, input/output token count, latency, cost. Track faithfulness và relevance scores trên sample of production queries. Alert khi error rate hoặc cost spike. Build dashboard (Grafana + OpenTelemetry — bạn đã biết) để visualize trends.

**Q: Làm sao handle khi LLM trả về sai format JSON?**
A: (1) Retry tối đa 3 lần, pass error message vào context ("Your previous response failed JSON parsing: ..."). (2) Dùng tool calling thay vì ask for JSON — tool calling reliable hơn. (3) Partial parsing — extract whatever is valid. (4) Fallback to default value nếu parsing hoàn toàn fail.
