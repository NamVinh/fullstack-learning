# Phase 06 — Agentic AI

> Agent = LLM có khả năng quyết định và thực thi nhiều bước tự động — không chỉ
> trả lời 1 câu hỏi mà còn gọi tools, read results, quyết định bước tiếp theo,
> lặp lại cho đến khi hoàn thành task.

---

## 1. Agent là gì — so với RAG thuần

```
RAG (stateless):
  User query → Retrieve → Generate → Done

Agent (stateful, multi-step):
  User task → Plan → [Tool call → Observe → Reason → Tool call → ...] → Final answer

Ví dụ Insurance Agent:
  Task: "Analyze claim #12345 and determine if it should be approved"

  Step 1: call get_claim_details(claimId: "12345")
  Step 2: call get_customer_history(customerId: "C-456")
  Step 3: call check_policy_coverage(policyId: "POL-789", claimType: "auto")
  Step 4: call search_fraud_patterns(claimSignatures: [...])
  Step 5: Generate final recommendation with all gathered data
```

---

## 2. Tool Calling — Core của Agent

### Define Tools (Anthropic)

```typescript
import Anthropic from "@anthropic-ai/sdk";

const INSURANCE_TOOLS: Anthropic.Tool[] = [
  {
    name: "get_claim_details",
    description: "Retrieve details of an insurance claim by claim ID",
    input_schema: {
      type: "object" as const,
      properties: {
        claim_id: {
          type: "string",
          description: "The unique claim identifier (e.g., CLM-2024-001)",
        },
      },
      required: ["claim_id"],
    },
  },
  {
    name: "get_customer_history",
    description: "Get customer's claim history and risk profile",
    input_schema: {
      type: "object" as const,
      properties: {
        customer_id: { type: "string" },
        months_back: {
          type: "number",
          description: "Number of months of history to retrieve",
          default: 24,
        },
      },
      required: ["customer_id"],
    },
  },
  {
    name: "check_fwa_patterns",
    description: "Check claim against known Fraud/Waste/Abuse patterns",
    input_schema: {
      type: "object" as const,
      properties: {
        claim_data: {
          type: "object",
          description: "Claim details to check against FWA patterns",
        },
      },
      required: ["claim_data"],
    },
  },
  {
    name: "update_claim_status",
    description: "Update the status of a claim in the system",
    input_schema: {
      type: "object" as const,
      properties: {
        claim_id: { type: "string" },
        status: {
          type: "string",
          enum: ["APPROVED", "REJECTED", "PENDING_INVESTIGATION", "PENDING_DOCS"],
        },
        reason: { type: "string" },
      },
      required: ["claim_id", "status", "reason"],
    },
  },
];
```

### Tool Executor

```typescript
// Map tool name to actual function
class InsuranceToolExecutor {
  constructor(
    private claimRepo: ClaimRepository,
    private customerRepo: CustomerRepository,
    private fwaEngine: FWAEngine
  ) {}

  async execute(toolName: string, toolInput: Record<string, any>): Promise<string> {
    switch (toolName) {
      case "get_claim_details":
        const claim = await this.claimRepo.findById(toolInput.claim_id);
        return JSON.stringify(claim);

      case "get_customer_history":
        const history = await this.customerRepo.getHistory(
          toolInput.customer_id,
          toolInput.months_back ?? 24
        );
        return JSON.stringify(history);

      case "check_fwa_patterns":
        const fwaResult = await this.fwaEngine.analyze(toolInput.claim_data);
        return JSON.stringify(fwaResult);

      case "update_claim_status":
        await this.claimRepo.updateStatus(
          toolInput.claim_id,
          toolInput.status,
          toolInput.reason
        );
        return JSON.stringify({ success: true });

      default:
        throw new Error(`Unknown tool: ${toolName}`);
    }
  }
}
```

---

## 3. Agent Loop — ReAct Pattern

```typescript
// ReAct = Reason + Act — model reason về kết quả, quyết định action tiếp theo

class InsuranceAgent {
  constructor(
    private client: Anthropic,
    private toolExecutor: InsuranceToolExecutor
  ) {}

  async run(task: string, maxSteps: number = 10): Promise<string> {
    const messages: Anthropic.MessageParam[] = [
      { role: "user", content: task },
    ];

    const systemPrompt = `
You are an insurance claims analyst agent. You have access to tools to:
- Retrieve claim and customer data
- Check FWA patterns
- Update claim status

Think step by step. Use tools to gather all necessary information before making decisions.
When you have enough information, provide your final analysis and recommendation.
Do NOT update claim status unless explicitly asked to do so.
`;

    let steps = 0;

    while (steps < maxSteps) {
      steps++;

      // 1. Call LLM
      const response = await this.client.messages.create({
        model: "claude-opus-4-8",  // Opus for complex reasoning
        max_tokens: 4096,
        system: systemPrompt,
        tools: INSURANCE_TOOLS,
        messages,
      });

      // 2. Check if done (no more tool calls)
      if (response.stop_reason === "end_turn") {
        const finalText = response.content
          .filter((b) => b.type === "text")
          .map((b) => b.text)
          .join("\n");
        return finalText;
      }

      // 3. Extract tool uses
      const toolUses = response.content.filter((b) => b.type === "tool_use");

      if (toolUses.length === 0) break;

      // 4. Add assistant message with tool use to history
      messages.push({ role: "assistant", content: response.content });

      // 5. Execute tools (can parallelize independent tools)
      const toolResults = await Promise.all(
        toolUses.map(async (toolUse) => {
          if (toolUse.type !== "tool_use") return null;

          try {
            const result = await this.toolExecutor.execute(
              toolUse.name,
              toolUse.input as Record<string, any>
            );
            return {
              type: "tool_result" as const,
              tool_use_id: toolUse.id,
              content: result,
            };
          } catch (err) {
            return {
              type: "tool_result" as const,
              tool_use_id: toolUse.id,
              is_error: true,
              content: `Error: ${err.message}`,
            };
          }
        })
      );

      // 6. Add tool results to history
      messages.push({
        role: "user",
        content: toolResults.filter(Boolean) as any,
      });
    }

    return "Agent reached maximum steps without completing the task.";
  }
}
```

---

## 4. Insurance-specific Agent Example

```typescript
// Claim analysis agent — full flow
const agent = new InsuranceAgent(client, toolExecutor);

const result = await agent.run(`
Analyze insurance claim CLM-2024-00456 for customer C-789.
Check their claim history, verify the claim details against FWA patterns,
and provide a detailed recommendation with risk score.
Do NOT update the claim status — this is for human review.
`);

// Agent sẽ tự động:
// 1. call get_claim_details("CLM-2024-00456")
// 2. call get_customer_history("C-789")
// 3. call check_fwa_patterns({ claim data from step 1 })
// 4. Synthesize all results → final recommendation với risk score
```

---

## 5. Memory Patterns

### Short-term memory (conversation history — bạn đã biết)

```typescript
// Conversation history đã covered ở Phase 02
// Agent cần quản lý history cẩn thận — có thể dài
```

### Long-term memory — Summarization

```typescript
// Khi history quá dài, summarize trước khi tiếp tục
async function summarizeHistory(
  history: Anthropic.MessageParam[]
): Promise<string> {
  const response = await client.messages.create({
    model: "claude-haiku-4-5",  // dùng model rẻ cho summarization
    max_tokens: 500,
    messages: [
      ...history,
      {
        role: "user",
        content:
          "Summarize the key findings and actions taken so far in 3-5 bullet points.",
      },
    ],
  });

  return response.content[0].type === "text" ? response.content[0].text : "";
}
```

### External memory (save to DB)

```typescript
// Lưu agent "memories" vào DB — retrieve khi cần
interface AgentMemory {
  agentId: string;
  customerId: string;
  fact: string;        // "Customer filed 3 claims in 2024"
  relevance: number;   // 0-1 — used for retrieval
  embedding: number[];
  createdAt: Date;
}
```

---

## 6. Human-in-the-Loop — quan trọng cho Insurance

Insurance cần human approval trước khi thực thi certain actions:

```typescript
class InsuranceAgentWithHITL extends InsuranceAgent {
  // Actions cần human approval
  private requiresApproval = ["update_claim_status", "trigger_investigation"];

  async run(task: string): Promise<AgentResult> {
    // ... normal agent loop, nhưng khi gặp sensitive tool:

    if (this.requiresApproval.includes(toolName)) {
      // Pause và ask for human approval
      const approval = await this.requestHumanApproval({
        action: toolName,
        parameters: toolInput,
        agentReasoning: "Agent determined claim is fraudulent based on...",
      });

      if (!approval.approved) {
        // Inject rejection into history và continue
        messages.push({
          role: "user",
          content: [{
            type: "tool_result",
            tool_use_id: toolUse.id,
            content: `Action rejected by human reviewer: ${approval.reason}`,
          }],
        });
        continue;
      }
    }

    // Execute normally if approved
    const result = await this.toolExecutor.execute(toolName, toolInput);
  }
}
```

---

## 7. Multi-Agent Patterns

```typescript
// Orchestrator → Specialist agents
class InsuranceOrchestrator {
  async processClaim(claimId: string): Promise<ClaimDecision> {
    // Parallel: run specialist agents concurrently
    const [fwaAnalysis, coverageCheck, documentValidation] = await Promise.all([
      this.fwaAgent.analyze(claimId),
      this.coverageAgent.verify(claimId),
      this.documentAgent.validate(claimId),
    ]);

    // Orchestrator synthesizes results
    return this.orchestratorLLM.synthesize({
      fwaAnalysis,
      coverageCheck,
      documentValidation,
    });
  }
}
```

---

## Câu hỏi phỏng vấn

**Q: Khi nào dùng agent vs RAG thuần?**
A: RAG đủ cho Q&A đơn giản (policy lookup, FAQ). Agent cần khi task multi-step và không biết trước cần data gì — ví dụ: "analyze claim và quyết định" cần gọi nhiều APIs theo thứ tự phụ thuộc kết quả của nhau.

**Q: Làm sao prevent agent loop vô hạn?**
A: maxSteps limit (10-20 steps thường đủ), timeout tổng, detect khi agent call cùng một tool với cùng arguments (infinite loop detection), budget token tracking.

**Q: Agent có reliable không — production ready chưa?**
A: Với task well-defined và tools rõ ràng — reliable. Với task ambiguous — vẫn cần human review. Trong insurance, pattern tốt nhất: agent thu thập thông tin và recommend, human approve final decision.
