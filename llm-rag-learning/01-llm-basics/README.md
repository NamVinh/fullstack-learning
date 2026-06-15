# Phase 01 — LLM Basics

> Hiểu cách LLM hoạt động ở mức đủ để build sản phẩm — không cần biết train model,
> nhưng phải hiểu tại sao model có behavior nhất định để debug và design tốt.

---

## 1. LLM là gì — góc nhìn của engineer

LLM (Large Language Model) là một hàm:

```
f(input_tokens) → probability distribution over next token
```

Mỗi lần generate, model chọn token tiếp theo dựa trên xác suất. Đây là lý do:
- Output không deterministic (trừ khi `temperature = 0`)
- Không có "memory" thực sự — mỗi request là stateless
- Không biết thông tin sau training cutoff
- Có thể "hallucinate" — generate text tự tin nhưng sai

**Điều quan trọng:** LLM không "biết" sự thật — nó predict text có xác suất cao nhất.

---

## 2. Kiến trúc Transformer (high-level, không cần toán)

```
Input text → Tokenizer → Token IDs
Token IDs → Embedding layer → Dense vectors
Dense vectors → N × [Attention + FFN] layers → Output logits
Output logits → Softmax → Next token probability
```

**Attention mechanism** là core: mỗi token "chú ý" đến các token khác trong context.
Đây là lý do LLM hiểu được ngữ cảnh dài.

**Bạn cần nhớ:**
- Transformer = self-attention + feed-forward layers, stacked N lần
- Larger model = more layers, more parameters = better reasoning (generally)
- Context window = số token tối đa model có thể "nhìn" trong một request

---

## 3. Tokenization

Token ≠ word. Tokenization phụ thuộc vào từng model:

```
"insurance" → ["insur", "ance"]       (2 tokens)
"claim"     → ["claim"]               (1 token)
"FWA"       → ["F", "W", "A"]         (3 tokens)
```

**Rule of thumb:** 1 token ≈ 0.75 word (English), tiếng Việt nhiều token hơn.

**Tại sao quan trọng:**
- Cost tính theo token (input + output)
- Context window limit tính theo token
- Truncation và chunking phải tính token, không phải character

```typescript
// Đếm token với tiktoken (OpenAI) hoặc @anthropic-ai/tokenizer
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();
const response = await client.messages.countTokens({
  model: "claude-opus-4-5",
  messages: [{ role: "user", content: "How many tokens is this?" }],
});
console.log(response.input_tokens); // → số token
```

---

## 4. Context Window

Context window = tổng token (system prompt + conversation history + output) model có thể xử lý.

| Model | Context Window |
|-------|---------------|
| Claude Sonnet 4.6 | 200,000 tokens |
| Claude Opus 4.8 | 200,000 tokens |
| GPT-4o | 128,000 tokens |
| GPT-4o-mini | 128,000 tokens |
| Llama 3.3 70B | 128,000 tokens |

**Ý nghĩa thực tế:**
- 200k tokens ≈ 150,000 words ≈ 500 trang sách — đủ để nhét nhiều document
- Nhưng cost tăng tuyến tính với input token — cần optimize
- Long context không phải lúc nào cũng tốt hơn RAG

---

## 5. Temperature và Sampling

```typescript
const response = await client.messages.create({
  model: "claude-sonnet-4-6",
  max_tokens: 1024,
  temperature: 0,      // 0 = deterministic, 1 = creative/random
  messages: [...]
});
```

| Temperature | Use case |
|-------------|----------|
| `0` | Extraction, classification, structured output — cần consistent |
| `0.3–0.5` | Summarization, translation — balance accuracy & fluency |
| `0.7–1.0` | Creative writing, brainstorming |

**Cho InsurTech:** Hầu hết tasks (claim extraction, fraud detection, document parsing) dùng `temperature: 0`.

---

## 6. Major Models — so sánh thực tế

### Anthropic Claude
```
claude-opus-4-8    → best reasoning, slowest, most expensive
claude-sonnet-4-6  → best balance (production default)
claude-haiku-4-5   → fastest, cheapest, simple tasks
```

### OpenAI
```
gpt-4o            → best multimodal (text + image + audio)
gpt-4o-mini       → fast, cheap, good for simple tasks
o3-mini           → strong reasoning (math, code, logic)
```

### Open Source (self-hosted)
```
Llama 3.3 70B     → best open source, comparable to GPT-4o-mini
Mistral Large     → good for European data compliance
Qwen2.5           → strong for Asian languages
```

**Cho InsurTech production:**
- Default: `claude-sonnet-4-6` (Anthropic) hoặc `gpt-4o` (OpenAI)
- Batch processing: `claude-haiku-4-5` hoặc `gpt-4o-mini`
- Reasoning tasks (fraud scoring): `claude-opus-4-8` hoặc `o3-mini`

---

## 7. API Structure cơ bản

### Anthropic Claude

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

const response = await client.messages.create({
  model: "claude-sonnet-4-6",
  max_tokens: 1024,
  system: "You are an insurance claims analyst.",  // system prompt
  messages: [
    { role: "user", content: "Analyze this claim: ..." },
    { role: "assistant", content: "Based on the claim..." },  // conversation history
    { role: "user", content: "What is the fraud risk score?" },
  ],
});

console.log(response.content[0].text);
// usage: { input_tokens: 150, output_tokens: 80 }
```

### OpenAI

```typescript
import OpenAI from "openai";

const client = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

const response = await client.chat.completions.create({
  model: "gpt-4o",
  messages: [
    { role: "system", content: "You are an insurance claims analyst." },
    { role: "user", content: "Analyze this claim: ..." },
  ],
  temperature: 0,
  max_tokens: 1024,
});

console.log(response.choices[0].message.content);
```

---

## 8. Streaming — quan trọng cho UX

Streaming trả về token theo thời gian thực — không phải đợi toàn bộ response.

```typescript
// Anthropic streaming
const stream = await client.messages.stream({
  model: "claude-sonnet-4-6",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Summarize this insurance policy..." }],
});

for await (const chunk of stream) {
  if (chunk.type === "content_block_delta" && chunk.delta.type === "text_delta") {
    process.stdout.write(chunk.delta.text);  // print as it streams
  }
}

const finalMessage = await stream.finalMessage();
```

```typescript
// Vercel AI SDK (Next.js) — bạn đã biết Next.js
import { streamText } from "ai";
import { anthropic } from "@ai-sdk/anthropic";

export async function POST(req: Request) {
  const { messages } = await req.json();
  const result = streamText({
    model: anthropic("claude-sonnet-4-6"),
    messages,
  });
  return result.toDataStreamResponse();
}
```

---

## 9. Multimodal — Text + Image (quan trọng cho Insurance)

Insurance documents thường là PDF/image. Models như Claude và GPT-4o đọc được image.

```typescript
// Gửi image cho Claude để extract thông tin từ claim document
const response = await client.messages.create({
  model: "claude-opus-4-8",
  max_tokens: 1024,
  messages: [{
    role: "user",
    content: [
      {
        type: "image",
        source: {
          type: "base64",
          media_type: "image/jpeg",
          data: base64ImageData,  // từ PDF page hoặc scanned document
        },
      },
      {
        type: "text",
        text: "Extract all fields from this insurance claim form as JSON.",
      },
    ],
  }],
});
```

---

## 10. Cost Estimation

```
Anthropic Claude Sonnet 4.6:
  Input:  $3 / 1M tokens
  Output: $15 / 1M tokens

OpenAI GPT-4o:
  Input:  $2.50 / 1M tokens
  Output: $10 / 1M tokens

Claude Haiku 4.5:
  Input:  $0.80 / 1M tokens
  Output: $4 / 1M tokens
```

**Tính cost cho RAG:**
- Query = ~500 token input + retrieved context ~2000 token = 2500 token input
- Response = ~500 token output
- Tổng 1 query với Sonnet: 2500 × $3/1M + 500 × $15/1M = $0.0075 + $0.0075 = **~$0.015/query**
- 10,000 queries/day = ~$150/day → cần optimize bằng caching và model routing

---

## Câu hỏi phỏng vấn thường gặp

**Q: LLM khác với database truyền thống như thế nào?**
A: Database lưu và retrieve thông tin chính xác. LLM generate text dựa trên patterns học được — không có "lưu trữ" fact, không đảm bảo accuracy, nhưng có khả năng reasoning và generation. Đó là lý do cần RAG để grounding LLM với dữ liệu thực tế.

**Q: Tại sao LLM hallucinate?**
A: LLM optimize để predict token có xác suất cao — không phải để verify truth. Khi không biết, model vẫn generate text tự tin vì đó là cách nó được train. Giải pháp: RAG (cung cấp context thực), structured output (force format), hoặc explicit "I don't know" prompting.

**Q: Context window lớn hơn có nghĩa là không cần RAG?**
A: Không hoàn toàn. Long context tốt cho single large document. Nhưng RAG hiệu quả hơn khi: (1) knowledge base lớn hơn context window, (2) cần real-time updated data, (3) muốn giảm cost bằng cách chỉ retrieve relevant chunks, (4) cần cite source cụ thể.
