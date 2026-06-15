# Phase 12 — Evaluation với RAGAS

> "You can't improve what you can't measure."
> Evaluation là thứ phân biệt production-ready RAG với demo RAG.
> Phase này cover RAGAS framework, cách build golden dataset,
> và integrate evaluation vào CI/CD pipeline.

---

## 1. Tại sao Evaluation quan trọng

```
Không có evaluation:
- Bạn không biết thay đổi prompt có cải thiện không
- Bạn không biết chunking strategy nào tốt hơn
- Bạn không phát hiện khi model hallucinate
- Bạn không thể A/B test các RAG configurations
- Bug trong production chỉ được phát hiện khi user complain

Có evaluation:
- Metrics cụ thể: "Faithfulness tăng từ 0.72 → 0.89 sau khi thêm reranking"
- Catch regression trước khi ship
- Data-driven decisions về kỹ thuật
- Automated testing trong CI/CD
```

---

## 2. RAGAS Framework — 4 Core Metrics

RAGAS = **RA**G **A**ssessment **S**ystem — framework Python để evaluate RAG pipelines.

### 4 Metrics chính

```
┌─────────────────────────────────────────────────────────┐
│                     RAG PIPELINE                        │
│                                                         │
│  Question ──► Retrieve ──► Generate ──► Answer          │
│      │            │                        │            │
│      │         Context                     │            │
│      └────────────┴────────────────────────┘            │
│                                                         │
│  Metrics:                                               │
│  1. Context Precision   = context relevant to question? │
│  2. Context Recall      = all needed info retrieved?    │
│  3. Answer Faithfulness = answer grounded in context?   │
│  4. Answer Relevance    = answer answers the question?  │
└─────────────────────────────────────────────────────────┘
```

### Metric 1: Context Precision

**Đo:** Trong các chunks được retrieve, bao nhiêu % thực sự relevant?

```
Context Precision = Relevant_chunks_retrieved / Total_chunks_retrieved

Ví dụ: Retrieve 5 chunks, 3 cái relevant → Precision = 3/5 = 0.6
Ví dụ tốt: Retrieve 3 chunks, 3 cái relevant → Precision = 3/3 = 1.0
```

Precision thấp → Nhiều noise trong context → LLM bị distract → Answer kém.

### Metric 2: Context Recall

**Đo:** Tất cả thông tin cần để answer có được retrieve không?

```
Context Recall = Relevant_info_retrieved / Total_relevant_info_in_KB

Cần "ground truth" — biết trước câu answer đúng cần những thông tin gì.
```

Recall thấp → Thiếu context → LLM phải hallucinate để fill gaps.

### Metric 3: Answer Faithfulness

**Đo:** Answer có bịa ra thông tin không có trong context không?

```
Faithfulness = Facts_supported_by_context / Total_facts_in_answer

Answer: "The deductible is $500 and the premium is $450/month"
Context: "The deductible is $500"  ← chỉ có deductible
→ $450 premium không có trong context → hallucination
→ Faithfulness = 1/2 = 0.5
```

### Metric 4: Answer Relevance

**Đo:** Answer có thực sự trả lời câu hỏi không?

```
Q: "What is covered under comprehensive auto insurance?"
A: "Auto insurance is important for vehicle owners." ← không answer question
→ Relevance thấp

Technique: Generate N questions từ answer, đo similarity với original question
```

---

## 3. Setup RAGAS

```bash
pip install ragas langchain-openai langchain-anthropic
```

```python
# Python (RAGAS là Python library)
from ragas import evaluate
from ragas.metrics import (
    faithfulness,
    answer_relevancy,
    context_precision,
    context_recall,
)
from datasets import Dataset
from langchain_anthropic import ChatAnthropic
from langchain_openai import OpenAIEmbeddings

# Data format RAGAS cần
evaluation_data = {
    "question": [
        "What is the deductible for auto claims under policy POL-001?",
        "Is flood damage covered in my property insurance?",
        "How long do I have to file a claim after an incident?",
    ],
    "answer": [
        # Answers từ RAG system của bạn
        "The deductible for auto claims is $500 per incident.",
        "Flood damage is not covered under the standard property policy. You need a separate flood insurance rider.",
        "Claims must be filed within 30 days of the incident date.",
    ],
    "contexts": [
        # Retrieved contexts cho mỗi question (list of strings)
        ["Section 4: Auto Coverage - Deductible amount: $500 per claim..."],
        ["Section 7: Property Coverage Exclusions - The following are not covered: flood, earthquake..."],
        ["Section 10: Claims Procedure - All claims must be reported within 30 calendar days..."],
    ],
    "ground_truth": [
        # Ground truth answer (cho context_recall)
        "The auto insurance deductible is $500.",
        "Standard policy does not cover flood damage; separate flood rider required.",
        "Claims must be filed within 30 days.",
    ],
}

dataset = Dataset.from_dict(evaluation_data)

# Configure với Claude
llm = ChatAnthropic(model="claude-sonnet-4-6", api_key=os.getenv("ANTHROPIC_API_KEY"))
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")

# Run evaluation
results = evaluate(
    dataset=dataset,
    metrics=[faithfulness, answer_relevancy, context_precision, context_recall],
    llm=llm,
    embeddings=embeddings,
)

print(results)
# {
#   "faithfulness": 0.92,
#   "answer_relevancy": 0.88,
#   "context_precision": 0.75,
#   "context_recall": 0.83,
# }
```

---

## 4. TypeScript: Collect Evaluation Data từ Production

```typescript
// Log RAG interactions để build evaluation dataset
interface RAGInteractionLog {
  id: string;
  timestamp: Date;
  question: string;
  retrievedContexts: string[];
  generatedAnswer: string;
  userId: string;
  sessionId: string;
  modelUsed: string;
  latencyMs: number;
  inputTokens: number;
  outputTokens: number;
}

@Injectable()
export class RAGEvaluationService {
  constructor(
    @InjectRepository(RAGLog) private logRepo: Repository<RAGLog>,
    private llm: LLMService,
  ) {}

  // Log mọi RAG interaction
  async logInteraction(data: RAGInteractionLog): Promise<void> {
    await this.logRepo.save(data);
  }

  // Sample production logs để create evaluation dataset
  async sampleForEvaluation(count: number = 100): Promise<RAGInteractionLog[]> {
    return this.logRepo.find({
      order: { timestamp: "DESC" },
      take: count,
      // Sample diverse questions, not just recent
    });
  }

  // LLM-as-Judge — evaluate individual answers
  async evaluateAnswer(
    question: string,
    context: string,
    answer: string
  ): Promise<{
    faithfulness: number;
    relevance: number;
    issues: string[];
  }> {
    const evaluation = await this.llm.completeWithTools(
      [{
        role: "user",
        content: `Evaluate this RAG system answer:

Question: ${question}

Retrieved Context:
${context}

Generated Answer:
${answer}

Evaluate:
1. Faithfulness (0-1): Does the answer only contain facts from the context?
2. Relevance (0-1): Does the answer actually address the question?
3. Issues: List any problems (hallucinations, missing info, off-topic content)`,
      }],
      "evaluate_rag_answer",
      {
        type: "object",
        properties: {
          faithfulness: { type: "number", minimum: 0, maximum: 1 },
          relevance: { type: "number", minimum: 0, maximum: 1 },
          issues: { type: "array", items: { type: "string" } },
        },
        required: ["faithfulness", "relevance", "issues"],
      }
    );

    return evaluation;
  }
}
```

---

## 5. Golden Dataset — Xương sống của Evaluation

```typescript
// Golden dataset = tập test cases với ground truth — không thay đổi
// Dùng để so sánh trước/sau khi thay đổi pipeline

interface GoldenTestCase {
  id: string;
  category: "coverage_query" | "claims_process" | "fwa_detection" | "policy_lookup";
  difficulty: "easy" | "medium" | "hard";
  question: string;
  policyNumber?: string;
  expectedKeywords: string[];    // answer PHẢI contain
  forbiddenContent: string[];    // answer KHÔNG ĐƯỢC contain
  groundTruthAnswer: string;
  groundTruthSource: string;     // exact quote từ document
  notes?: string;
}

// Ví dụ golden dataset cho insurance
const GOLDEN_DATASET: GoldenTestCase[] = [
  {
    id: "TC-001",
    category: "coverage_query",
    difficulty: "easy",
    question: "What is the auto collision deductible?",
    policyNumber: "POL-TEST-001",
    expectedKeywords: ["$500", "collision", "deductible"],
    forbiddenContent: ["comprehensive", "medical"],
    groundTruthAnswer: "The auto collision deductible is $500 per incident.",
    groundTruthSource: "Section 4.2: Collision Coverage - Deductible: $500",
  },
  {
    id: "TC-002",
    category: "coverage_query",
    difficulty: "medium",
    question: "If my car is stolen and I have $200 cash inside, will the cash be covered?",
    policyNumber: "POL-TEST-001",
    expectedKeywords: ["not covered", "cash", "personal property"],
    forbiddenContent: [],
    groundTruthAnswer: "Cash and currency are explicitly excluded from auto theft coverage. Only the vehicle itself and permanently installed equipment are covered.",
    groundTruthSource: "Section 4.5: Theft Coverage Exclusions - The following are not covered: money, securities, credit cards...",
  },
  {
    id: "TC-003",
    category: "claims_process",
    difficulty: "easy",
    question: "How many days do I have to report a claim?",
    policyNumber: "POL-TEST-001",
    expectedKeywords: ["30 days", "report", "notification"],
    forbiddenContent: [],
    groundTruthAnswer: "You must report the claim within 30 days of the incident.",
    groundTruthSource: "Section 8.1: Claims Notification - Claims must be reported within 30 calendar days...",
  },
  {
    id: "TC-004",
    category: "fwa_detection",
    difficulty: "hard",
    question: "Analyze this claim: Vehicle stolen 15 days after policy inception, $80,000 luxury car, no police report, customer's 4th claim this year.",
    expectedKeywords: ["high risk", "fraud", "investigate"],
    forbiddenContent: ["approve"],
    groundTruthAnswer: "HIGH RISK - Multiple fraud indicators: (1) Claim within 30-day inception period, (2) 4th claim this year - significantly above average, (3) No police report for theft claim is suspicious, (4) High vehicle value increases motive. Recommendation: INVESTIGATE",
    groundTruthSource: "FWA Pattern Library: Early Claim + High Frequency + High Value",
  },
];
```

---

## 6. Evaluation Runner — TypeScript

```typescript
// Run golden dataset evaluation và generate report
@Injectable()
export class EvaluationRunner {
  constructor(
    private ragService: RAGService,
    private evaluationService: RAGEvaluationService,
  ) {}

  async runGoldenDataset(): Promise<EvaluationReport> {
    const results: TestResult[] = [];

    for (const tc of GOLDEN_DATASET) {
      console.log(`Running: ${tc.id} - ${tc.question.slice(0, 50)}...`);

      // 1. Run RAG
      const { answer, contexts } = await this.ragService.queryWithContext(
        tc.question,
        tc.policyNumber ? { policyNumber: tc.policyNumber } : undefined
      );

      // 2. Check expected keywords
      const missingKeywords = tc.expectedKeywords.filter(
        (kw) => !answer.toLowerCase().includes(kw.toLowerCase())
      );

      // 3. Check forbidden content
      const foundForbidden = tc.forbiddenContent.filter(
        (f) => answer.toLowerCase().includes(f.toLowerCase())
      );

      // 4. LLM evaluation
      const llmEval = await this.evaluationService.evaluateAnswer(
        tc.question,
        contexts.join("\n\n"),
        answer
      );

      const passed =
        missingKeywords.length === 0 &&
        foundForbidden.length === 0 &&
        llmEval.faithfulness >= 0.8 &&
        llmEval.relevance >= 0.7;

      results.push({
        testId: tc.id,
        category: tc.category,
        difficulty: tc.difficulty,
        passed,
        answer,
        faithfulness: llmEval.faithfulness,
        relevance: llmEval.relevance,
        missingKeywords,
        foundForbidden,
        issues: llmEval.issues,
      });
    }

    return this.generateReport(results);
  }

  private generateReport(results: TestResult[]): EvaluationReport {
    const byCategory = GOLDEN_DATASET.reduce((acc, tc) => {
      if (!acc[tc.category]) acc[tc.category] = { total: 0, passed: 0 };
      acc[tc.category].total++;
      if (results.find((r) => r.testId === tc.id)?.passed) {
        acc[tc.category].passed++;
      }
      return acc;
    }, {} as Record<string, { total: number; passed: number }>);

    return {
      totalTests: results.length,
      passed: results.filter((r) => r.passed).length,
      passRate: results.filter((r) => r.passed).length / results.length,
      avgFaithfulness: results.reduce((s, r) => s + r.faithfulness, 0) / results.length,
      avgRelevance: results.reduce((s, r) => s + r.relevance, 0) / results.length,
      byCategory,
      failedTests: results.filter((r) => !r.passed),
      timestamp: new Date(),
    };
  }
}
```

---

## 7. CI/CD Integration

```yaml
# .github/workflows/rag-evaluation.yml
name: RAG Evaluation

on:
  pull_request:
    paths:
      - 'src/ai/**'           # trigger khi AI code thay đổi
      - 'prompts/**'           # trigger khi prompts thay đổi
  schedule:
    - cron: '0 9 * * 1'       # Monday 9 AM — weekly regression check

jobs:
  evaluate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Run RAG Evaluation
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          DATABASE_URL: ${{ secrets.TEST_DATABASE_URL }}
          QDRANT_URL: ${{ secrets.TEST_QDRANT_URL }}
        run: npm run evaluate:rag

      - name: Check evaluation thresholds
        run: |
          node -e "
            const report = require('./eval-results.json');
            const THRESHOLDS = {
              passRate: 0.85,
              avgFaithfulness: 0.80,
              avgRelevance: 0.75,
            };
            const failures = Object.entries(THRESHOLDS).filter(
              ([key, threshold]) => report[key] < threshold
            );
            if (failures.length > 0) {
              console.error('EVALUATION FAILED:');
              failures.forEach(([key, threshold]) => {
                console.error(\`  \${key}: \${report[key].toFixed(2)} < \${threshold}\`);
              });
              process.exit(1);
            }
            console.log('All evaluation thresholds passed!');
          "

      - name: Comment PR with evaluation results
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            const report = require('./eval-results.json');
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## RAG Evaluation Results
              
              | Metric | Score | Threshold | Status |
              |--------|-------|-----------|--------|
              | Pass Rate | ${(report.passRate * 100).toFixed(1)}% | 85% | ${report.passRate >= 0.85 ? '✅' : '❌'} |
              | Faithfulness | ${report.avgFaithfulness.toFixed(2)} | 0.80 | ${report.avgFaithfulness >= 0.80 ? '✅' : '❌'} |
              | Relevance | ${report.avgRelevance.toFixed(2)} | 0.75 | ${report.avgRelevance >= 0.75 ? '✅' : '❌'} |
              
              ${report.failedTests.length > 0 ? `**Failed tests:** ${report.failedTests.map(t => t.testId).join(', ')}` : '**All tests passed!**'}`
            });
```

---

## 8. A/B Testing RAG Configurations

```typescript
// So sánh 2 RAG configurations systematically
interface RAGConfig {
  name: string;
  chunkSize: number;
  chunkOverlap: number;
  topK: number;
  useReranking: boolean;
  embeddingModel: string;
  llmModel: string;
}

async function abTestConfigs(
  configA: RAGConfig,
  configB: RAGConfig,
  dataset: GoldenTestCase[]
): Promise<ABTestResult> {
  const [resultsA, resultsB] = await Promise.all([
    runEvalWithConfig(configA, dataset),
    runEvalWithConfig(configB, dataset),
  ]);

  return {
    configA: { name: configA.name, ...resultsA },
    configB: { name: configB.name, ...resultsB },
    winner: resultsA.passRate >= resultsB.passRate ? configA.name : configB.name,
    improvement: {
      passRate: resultsB.passRate - resultsA.passRate,
      faithfulness: resultsB.avgFaithfulness - resultsA.avgFaithfulness,
      relevance: resultsB.avgRelevance - resultsA.avgRelevance,
    },
  };
}

// Ví dụ A/B test:
const result = await abTestConfigs(
  { name: "baseline", chunkSize: 1000, topK: 5, useReranking: false, ... },
  { name: "with_reranking", chunkSize: 1000, topK: 5, useReranking: true, ... },
  GOLDEN_DATASET
);

// Output:
// winner: "with_reranking"
// improvement: { passRate: +0.12, faithfulness: +0.08, relevance: +0.05 }
// → Reranking cải thiện pass rate 12%
```

---

## Câu hỏi phỏng vấn

**Q: Làm sao build golden dataset tốt?**
A: (1) Cover diverse categories — không phải chỉ easy questions. (2) Include edge cases — ambiguous queries, questions spanning multiple documents, negative tests ("is X covered?" khi X không được covered). (3) Annotate bởi domain expert (insurance analyst, không phải developer). (4) Verify ground truth bằng cách tìm exact quote từ source document. (5) Review và update khi documents thay đổi. Bắt đầu với 20-30 cases chất lượng cao, scale lên theo thời gian.

**Q: Evaluation thresholds nên set ở mức nào?**
A: Depends on use case risk. Customer-facing Q&A: Faithfulness ≥ 0.85, Relevance ≥ 0.80. FWA detection (high-stakes): Faithfulness ≥ 0.95, pass rate ≥ 0.90 (cost of false positive = miss real fraud). Document extraction: Precision ≥ 0.90 (wrong data causes downstream errors). Start conservative, relax nếu threshold quá strict cho timeline.
