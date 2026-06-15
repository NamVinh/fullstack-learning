# Phase 08 — InsurTech Context

> Apply các kỹ thuật từ Phase 01–07 vào domain cụ thể của JD: Life/Non-life insurance,
> Claim, Underwriting, Policy Servicing, và FWA (Fraud/Waste/Abuse) detection.
> Hiểu domain giúp bạn hỏi đúng câu hỏi và propose đúng solution trong interview.

---

## 1. Insurance Domain 101

### Các loại insurance

```
Life Insurance:
  - Term Life: bảo hiểm tử vong trong một khoảng thời gian
  - Whole Life: bảo hiểm tử vong suốt đời, có cash value
  - Endowment: vừa bảo hiểm vừa tiết kiệm

Non-Life Insurance:
  - Auto: xe cơ giới
  - Property: nhà, tài sản
  - Health: y tế (đôi khi tách riêng)
  - Travel: du lịch
  - Liability: trách nhiệm dân sự
```

### Các concept cốt lõi

```
Policy:       Hợp đồng bảo hiểm — quyền lợi, exclusions, premium, period
Premium:      Phí bảo hiểm khách hàng đóng
Claim:        Yêu cầu bồi thường khi xảy ra sự kiện được bảo hiểm
Deductible:   Số tiền khách hàng tự chịu trước khi bảo hiểm trả
Coverage:     Phạm vi bảo hiểm — gì được bảo hiểm và giới hạn bao nhiêu
Exclusion:    Những trường hợp không được bảo hiểm
Underwriting: Quá trình đánh giá rủi ro để quyết định accept/reject và premium
FWA:          Fraud/Waste/Abuse — gian lận, lãng phí, lạm dụng
```

### Claim lifecycle

```
1. FNOL (First Notice of Loss): khách hàng báo sự cố
2. Assignment: assign claim adjuster
3. Investigation: thu thập thông tin, documents
4. Evaluation: assess validity và coverage
5. Decision: approve/reject/partial
6. Payment: giải quyết bồi thường
7. Closure: đóng claim
```

---

## 2. AI Use Cases trong Insurance

### Use case 1: Document Processing & Extraction

```
Input: PDF claim form, scanned documents, medical records, police reports
Output: Structured data (JSON)

Tech: LLM multimodal (Claude/GPT-4o vision) + PDF parsing
```

```typescript
const DOCUMENT_EXTRACTION_PROMPT = `
You are an insurance document processing system.
Extract all information from the provided document and return as JSON.

For claim forms extract:
- Claimant information (name, DOB, policy number, contact)
- Incident details (date, time, location, description)
- Damages/losses (itemized list với estimated values)
- Witness information if present
- Signatures and dates

Rules:
- Extract ONLY what is explicitly stated
- Use null for missing fields
- Preserve exact monetary values, dates, policy numbers
- Flag any illegible sections as "ILLEGIBLE"
`;
```

### Use case 2: FWA Detection

```
FWA = Fraud/Waste/Abuse — Core use case của JD này

Fraud signals:
- Claim filed shortly after policy inception
- Multiple claims in short period
- High claim value relative to policy history
- Inconsistent information (date, location, witnesses)
- Known fraud patterns (staged accidents, inflated invoices)
- Social network analysis (connections to known fraudsters)

Waste signals:
- Unnecessary procedures/treatments
- Billing for services not rendered
- Duplicate billing

Abuse signals:
- Excessive claim frequency
- Claims near policy limits repeatedly
- Pattern matches known abuse schemes
```

```typescript
interface FWAAnalysisResult {
  overallRiskScore: number;        // 0-100
  riskLevel: "LOW" | "MEDIUM" | "HIGH" | "CRITICAL";
  fraudSignals: FraudSignal[];
  wasteSignals: WasteSignal[];
  abuseSignals: AbuseSignal[];
  recommendation: "APPROVE" | "INVESTIGATE" | "REJECT" | "REQUEST_ADDITIONAL_DOCS";
  reasoning: string;
  confidence: number;
  requiredActions?: string[];
}

interface FraudSignal {
  type: string;
  description: string;
  severity: "LOW" | "MEDIUM" | "HIGH";
  evidence: string;
}

const FWA_ANALYSIS_PROMPT = `
You are a senior insurance fraud analyst with expertise in FWA detection.
Analyze the following claim and customer data for fraud, waste, and abuse indicators.

Known FWA patterns to check:
FRAUD:
- Claim filed within 30 days of policy inception
- More than 3 claims in 12 months
- Prior fraud investigation history
- Inconsistent incident details across documents
- Inflated or unsupported damage values
- Missing or suspicious documentation
- High-value claim on recently increased coverage

WASTE:
- Claimed items significantly above market value
- Services duplicated or not typically necessary
- Provider is known for overbilling

ABUSE:
- Claim frequency significantly above demographic average
- Pattern of claims near renewal periods
- Claims that consistently reach policy limits

Analyze each category and provide a structured risk assessment.
`;

// FWA Engine với LLM + Rules
@Injectable()
export class FWAEngineService {
  constructor(
    private llm: LLMService,
    private claimRepo: ClaimRepository,
    private ruleEngine: RuleEngineService  // deterministic rules chạy TRƯỚC LLM
  ) {}

  async analyze(claimId: string): Promise<FWAAnalysisResult> {
    // 1. Gather data
    const claim = await this.claimRepo.findWithFullHistory(claimId);
    const claimHistory = await this.claimRepo.findByCustomer(claim.customerId, { months: 24 });
    const policyDetails = await this.policyRepo.findById(claim.policyId);

    // 2. Run deterministic rules first (fast, free)
    const ruleFlags = await this.ruleEngine.evaluate(claim, claimHistory, policyDetails);

    // 3. If clear (no rule flags), approve quickly without LLM call
    if (ruleFlags.length === 0 && claim.amount < 1000) {
      return { overallRiskScore: 5, riskLevel: "LOW", recommendation: "APPROVE", ... };
    }

    // 4. Use LLM for nuanced analysis
    const context = this.buildFWAContext(claim, claimHistory, policyDetails, ruleFlags);

    const result = await this.llm.completeWithTools<FWAAnalysisResult>(
      [{ role: "user", content: context }],
      "fwa_analysis",
      FWA_RESULT_SCHEMA,
      { system: FWA_ANALYSIS_PROMPT, temperature: 0 }
    );

    return result;
  }

  private buildFWAContext(claim: Claim, history: Claim[], policy: Policy, flags: RuleFlag[]): string {
    return `
CLAIM DETAILS:
${JSON.stringify(claim, null, 2)}

CUSTOMER CLAIM HISTORY (last 24 months):
Total claims: ${history.length}
Total amount: $${history.reduce((s, c) => s + c.amount, 0)}
Claims: ${JSON.stringify(history.map(c => ({ date: c.date, type: c.type, amount: c.amount, status: c.status })), null, 2)}

POLICY DETAILS:
Inception date: ${policy.inceptionDate}
Coverage amount: $${policy.coverageAmount}
Type: ${policy.type}

PRE-SCREENING FLAGS (automated rules):
${flags.length === 0 ? "None" : flags.map(f => `- ${f.type}: ${f.description}`).join("\n")}

Analyze for FWA risk.
`;
  }
}
```

### Use case 3: Underwriting Assistant

```typescript
// Underwriting = đánh giá rủi ro khi accept policy
// AI assist: extract risk factors, suggest premium adjustments, check eligibility

const UNDERWRITING_PROMPT = `
You are an underwriting assistant for a non-life insurance company.
Evaluate the insurance application and provide a risk assessment.

Assess:
1. Risk factors present in the application
2. Coverage appropriateness for stated values
3. Missing information that must be obtained
4. Recommended premium adjustment (if any)
5. Conditions or exclusions that should apply
6. Accept/Decline recommendation with justification

Do NOT make final decisions — provide analysis for human underwriter review.
`;
```

### Use case 4: Policy Q&A Chatbot (RAG)

```typescript
// Customer asks about their coverage — RAG retrieves from policy document

const policyQASystem = async (customerId: string, question: string) => {
  // Get customer's active policies
  const policies = await policyRepo.findActiveByCustomer(customerId);

  // Retrieve relevant policy sections
  const chunks = await vectorStore.search(
    await embedding.embed(question),
    5,
    { customerId, documentType: "policy" }
  );

  // Generate answer grounded in policy
  return ragService.query(question, {
    systemPrompt: `
You are an insurance policy assistant helping customer ${customerId}.
Answer questions based ONLY on their policy documents.
Be specific about coverage amounts, deductibles, and exclusions.
If something is not covered, clearly state that and explain why.
Always cite the relevant policy section.
`,
    topK: 5,
    filter: { customerId },
  });
};
```

---

## 3. Architecture cho InsurTech GenAI System

```
┌─────────────────────────────────────────────────────────────────┐
│                    InsurTech GenAI Platform                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Frontend (Next.js)                                             │
│  ├─ Claims Portal (submit, track, chat)                        │
│  ├─ Agent Dashboard (underwriter workbench)                    │
│  └─ Admin Panel (FWA reports, model monitoring)                │
│                                                                 │
│  Backend (NestJS)                                               │
│  ├─ Document Service (PDF ingestion, OCR, extraction)          │
│  ├─ RAG Service (policy Q&A, claim lookup)                     │
│  ├─ FWA Engine (fraud detection, risk scoring)                 │
│  ├─ Underwriting Service (risk assessment)                     │
│  └─ LLM Gateway (rate limiting, caching, logging, routing)     │
│                                                                 │
│  Data Layer                                                     │
│  ├─ PostgreSQL + pgvector (policies, claims, embeddings)       │
│  ├─ Redis (caching, sessions, rate limiting)                   │
│  └─ Object Storage (document files)                            │
│                                                                 │
│  AI Layer                                                       │
│  ├─ Claude Sonnet 4.6 (default — RAG, extraction)             │
│  ├─ Claude Opus 4.8 (complex reasoning — FWA analysis)        │
│  └─ Claude Haiku 4.5 (safety checks, simple tasks)            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Compliance và Data Privacy

```typescript
// Insurance data là extremely sensitive — MUST consider:

// 1. PII handling — không log raw data
const sanitizeForLogging = (data: any): any => {
  const sensitiveFields = ["name", "dob", "ssn", "address", "phone", "email", "medicalRecord"];
  return Object.fromEntries(
    Object.entries(data).map(([k, v]) => [
      k,
      sensitiveFields.includes(k) ? "[REDACTED]" : v,
    ])
  );
};

// 2. Data residency — một số quốc gia yêu cầu data không rời khỏi country
// → Consider self-hosted models (Llama, Mistral) hoặc regional API endpoints

// 3. Audit trail — mọi AI decision phải auditable
interface AIDecisionLog {
  decisionId: string;
  claimId: string;
  model: string;
  inputHash: string;    // hash of input, not full input (privacy)
  output: string;
  confidence: number;
  humanReviewed: boolean;
  reviewerId?: string;
  timestamp: Date;
}

// 4. Explainability — AI decision phải explainable
// → Luôn include reasoning trong output schema
// → Human reviewer phải hiểu tại sao AI recommend gì
```

---

## 5. Câu hỏi phỏng vấn InsurTech cụ thể

**Q: Làm sao build FWA engine với AI?**
A: Hybrid approach: (1) Deterministic rule engine chạy trước — nhanh, rẻ, cover clear-cut cases (claim < 30 ngày sau inception = flag). (2) LLM analysis cho nuanced patterns — analyze context, cross-reference history, reason about inconsistencies. (3) Human review cho high-risk cases (score > 70). Output phải include structured reasoning để human reviewer có thể audit và override.

**Q: RAG cho insurance documents có challenges gì?**
A: (1) Policy documents dùng legal/technical language — embedding quality thấp hơn, cần domain-specific embedding model hoặc fine-tuning. (2) Policy versions — customer hỏi về policy đang áp dụng, không phải version cũ. Cần filter bằng policy effective date. (3) Tables và structured data trong PDF — cần đặc biệt handle, không phải plain text chunking. (4) Cross-document reference — policy refer đến endorsement, schedule — cần link aware chunking.

**Q: Làm sao handle khi AI đưa ra quyết định sai trong insurance context?**
A: (1) Never fully automate high-stakes decisions — FWA above threshold hoặc claim above certain amount luôn cần human review. (2) Confidence scoring — nếu AI confidence thấp → escalate to human. (3) Audit trail đầy đủ — mọi AI recommendation được log với reasoning để investigate sau. (4) Feedback loop — khi human override AI, record đó để improve model/prompts. (5) Regular evaluation trên golden dataset để detect degradation.

**Q: Bạn sẽ design document indexing pipeline như thế nào?**
A: (1) Upload trigger → queue job (không process sync trong request). (2) Extract text: PDF → pdf-parse, scanned images → OCR (AWS Textract hoặc Google Document AI). (3) Pre-process: clean whitespace, identify document sections. (4) Chunk theo section boundaries khi có (không phải fixed size). (5) Embed batch với text-embedding-3-small. (6) Store vào pgvector với metadata (documentId, policyNumber, customerId, version, effectiveDate). (7) Confirm completion và notify via webhook/event.
