# Phase 09 — Fine-Tuning

> Fine-tuning = tiếp tục train một pre-trained model trên dataset của riêng bạn
> để model học behavior, style, hoặc domain knowledge cụ thể.
> Đây KHÔNG phải train from scratch — bạn đang "tinh chỉnh" model đã có.

---

## 1. Khi nào nên Fine-tune? — Decision Tree

```
Bạn có vấn đề gì?
│
├─ Model không biết domain knowledge của tôi (insurance terms, internal docs)
│   └─ → Dùng RAG trước. Fine-tune chỉ khi RAG không đủ.
│
├─ Model trả lời đúng nhưng sai format/style (tôi muốn JSON không có ```json wrapper)
│   └─ → Fine-tune là đúng. RAG không fix được format behavior.
│
├─ Tôi cần latency thấp hơn / cost thấp hơn cho task specific
│   └─ → Fine-tune model nhỏ hơn cho task đó (ví dụ: fine-tune Haiku cho classification)
│
├─ Task rất specific và repeated (claim classification, entity extraction)
│   └─ → Fine-tune sau khi đã có 100+ labeled examples
│
└─ Tôi chỉ cần model "nhớ" thông tin công ty
    └─ → KHÔNG fine-tune. Dùng RAG + system prompt.
```

**Quy tắc vàng:**
```
Few-shot prompting → RAG → Fine-tuning → Train from scratch
(rẻ, nhanh)                              (đắt, chậm)
```

Thử theo thứ tự này. Chỉ next step khi current step không đủ.

---

## 2. Các loại Fine-tuning

### Full Fine-tuning
- Update **toàn bộ** parameters của model
- Kết quả tốt nhất nhưng **rất đắt** (cần GPU cluster)
- Chỉ feasible với open-source model tự host
- Không practical cho hầu hết startup

### LoRA / QLoRA — Low-Rank Adaptation
```
Thay vì update toàn bộ weight matrix W (rất lớn):
W_new = W_original + ΔW

LoRA decompose ΔW thành 2 ma trận nhỏ:
ΔW = A × B  (A: d×r, B: r×k, với r << d,k)

→ Chỉ train A và B — tiết kiệm memory 10-100x
→ Kết quả tương đương full fine-tuning cho nhiều tasks
```

QLoRA = LoRA + quantization (4-bit) → chạy được trên single consumer GPU (24GB VRAM).

### Instruction Fine-tuning (SFT)
- Train model để follow instructions tốt hơn
- Format: `{"instruction": "...", "input": "...", "output": "..."}`
- Đây là cách tạo ra các instruction-following models như Claude, ChatGPT

### RLHF — Reinforcement Learning from Human Feedback
- Dùng human preference để train reward model
- Sau đó dùng RL (PPO) để optimize LLM theo reward model
- Đây là cách Anthropic train Claude
- **Không practical** cho individual developer

---

## 3. OpenAI Fine-tuning API — Dễ nhất cho Production

OpenAI cung cấp managed fine-tuning — không cần GPU, không cần infra.

### Supported models
```
gpt-4o-mini-2024-07-18   ← Recommended: best cost/quality ratio
gpt-3.5-turbo-0125
gpt-4o-2024-08-06
```

### Step 1: Prepare Training Data

```typescript
// Format: JSONL (JSON Lines) — mỗi line là một example
// File: training_data.jsonl

const trainingExamples = [
  {
    messages: [
      { role: "system", content: "You are an insurance claim classifier." },
      { role: "user", content: "Windshield cracked by flying rock on highway." },
      { role: "assistant", content: JSON.stringify({
        claimType: "auto",
        subType: "comprehensive",
        damageCategory: "glass",
        urgency: "low",
        estimatedAmount: "200-500"
      })},
    ],
  },
  {
    messages: [
      { role: "system", content: "You are an insurance claim classifier." },
      { role: "user", content: "House flooded after heavy rain, ground floor damaged." },
      { role: "assistant", content: JSON.stringify({
        claimType: "property",
        subType: "water_damage",
        damageCategory: "structural",
        urgency: "high",
        estimatedAmount: "5000-20000"
      })},
    ],
  },
  // Cần tối thiểu 10 examples, recommended 50-100+
];

// Convert to JSONL
import fs from "fs";
const jsonl = trainingExamples.map((e) => JSON.stringify(e)).join("\n");
fs.writeFileSync("training_data.jsonl", jsonl);
```

### Step 2: Validate Data

```typescript
import OpenAI from "openai";

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

// Upload training file
const file = await openai.files.create({
  file: fs.createReadStream("training_data.jsonl"),
  purpose: "fine-tune",
});

console.log("File uploaded:", file.id);  // file-abc123
```

### Step 3: Create Fine-tuning Job

```typescript
const job = await openai.fineTuning.jobs.create({
  training_file: file.id,
  model: "gpt-4o-mini-2024-07-18",
  hyperparameters: {
    n_epochs: 3,          // số lần train qua toàn bộ dataset (3-10)
    batch_size: "auto",
    learning_rate_multiplier: "auto",
  },
  // Optional: validation file để track overfitting
  // validation_file: validationFileId,
  suffix: "insurance-classifier",  // model name suffix
});

console.log("Job created:", job.id);  // ftjob-abc123
```

### Step 4: Monitor Progress

```typescript
// Poll status
const checkStatus = async (jobId: string) => {
  const job = await openai.fineTuning.jobs.retrieve(jobId);
  console.log("Status:", job.status);  // queued → running → succeeded/failed
  console.log("Trained tokens:", job.trained_tokens);
  console.log("Fine-tuned model:", job.fine_tuned_model);
  // → ft:gpt-4o-mini-2024-07-18:your-org:insurance-classifier:abc123
};

// List events (training progress)
const events = await openai.fineTuning.jobs.listEvents(job.id);
events.data.forEach((e) => console.log(e.created_at, e.message));
// "Step 100/300: training loss=0.45"
// "Step 200/300: training loss=0.23"
```

### Step 5: Use Fine-tuned Model

```typescript
// Dùng y như model bình thường, chỉ thay model name
const response = await openai.chat.completions.create({
  model: "ft:gpt-4o-mini-2024-07-18:your-org:insurance-classifier:abc123",
  messages: [
    { role: "system", content: "You are an insurance claim classifier." },
    { role: "user", content: "Car engine seized due to lack of oil maintenance." },
  ],
  temperature: 0,
});

console.log(response.choices[0].message.content);
// → {"claimType":"auto","subType":"mechanical","damageCategory":"engine",...}
```

---

## 4. Anthropic Fine-tuning

Anthropic fine-tuning hiện tại available qua API (Claude Haiku models):

```typescript
// Anthropic fine-tuning — similar concept, khác API
// Documentation: docs.anthropic.com/en/docs/build-with-claude/fine-tuning

// Format training data (Anthropic format)
const anthropicTrainingExample = {
  messages: [
    { role: "user", content: "Classify this claim: water leak from upstairs neighbor" },
    { role: "assistant", content: '{"claimType":"property","cause":"water_damage","liability":"third_party"}' },
  ],
};
```

---

## 5. Open-Source Fine-tuning với LoRA (Self-hosted)

Khi cần: data privacy (không muốn gửi data cho OpenAI/Anthropic), custom base model, maximum control.

### Tools cần biết

```
Hugging Face Transformers  — load và train models
PEFT (Parameter-Efficient Fine-Tuning)  — LoRA implementation
TRL (Transformer Reinforcement Learning)  — SFT Trainer
bitsandbytes  — quantization (QLoRA)
```

### QLoRA Example (Llama 3.1 8B)

```python
# Python (không phải TypeScript — fine-tuning thường dùng Python)
from transformers import AutoModelForCausalLM, AutoTokenizer, BitsAndBytesConfig
from peft import LoraConfig, get_peft_model, TaskType
from trl import SFTTrainer, SFTConfig

# 1. Load model với 4-bit quantization (QLoRA)
bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype="bfloat16",
)

model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Meta-Llama-3.1-8B-Instruct",
    quantization_config=bnb_config,
    device_map="auto",
)
tokenizer = AutoTokenizer.from_pretrained("meta-llama/Meta-Llama-3.1-8B-Instruct")

# 2. LoRA config
lora_config = LoraConfig(
    r=16,                    # rank — higher = more capacity, more memory
    lora_alpha=32,           # scaling factor
    target_modules=["q_proj", "v_proj"],  # which layers to adapt
    lora_dropout=0.05,
    bias="none",
    task_type=TaskType.CAUSAL_LM,
)

model = get_peft_model(model, lora_config)
model.print_trainable_parameters()
# trainable params: 3,407,872 || all params: 8,033,669,120 || trainable%: 0.04

# 3. Train
trainer = SFTTrainer(
    model=model,
    train_dataset=your_dataset,
    args=SFTConfig(
        output_dir="./insurance-llama",
        num_train_epochs=3,
        per_device_train_batch_size=4,
        gradient_accumulation_steps=4,
        learning_rate=2e-4,
        fp16=True,
        logging_steps=10,
    ),
)

trainer.train()

# 4. Save & merge LoRA weights
model.save_pretrained("./insurance-llama-lora")
# Merge và upload to Hugging Face Hub để dùng trong production
```

### Deploy fine-tuned model

```typescript
// Option 1: Hugging Face Inference API
const response = await fetch(
  "https://api-inference.huggingface.co/models/your-org/insurance-llama",
  {
    method: "POST",
    headers: { Authorization: `Bearer ${process.env.HF_TOKEN}` },
    body: JSON.stringify({ inputs: "Classify this claim: ..." }),
  }
);

// Option 2: Ollama (local, for development)
// ollama run your-fine-tuned-model

// Option 3: vLLM (production, self-hosted)
// Highly optimized inference server cho open-source models
```

---

## 6. Fine-tuning vs RAG — Bảng so sánh đầy đủ

| | Fine-tuning | RAG |
|--|-------------|-----|
| **Knowledge update** | Cần retrain khi data thay đổi | Real-time, chỉ update vector DB |
| **Setup cost** | Cao (data preparation, training time) | Thấp hơn |
| **Inference cost** | Thấp (ít token hơn vì không cần inject context) | Cao hơn (retrieval + large context) |
| **Accuracy cho facts** | Có thể hallucinate | Grounded trong documents |
| **Source citation** | Không thể cite | Có thể cite exact source |
| **Style/format** | Rất tốt | Kém hơn |
| **Task-specific behavior** | Rất tốt | Trung bình |
| **Data privacy** | Data expose cho provider (trừ self-hosted) | Data chỉ trong context |
| **Best for** | Format, style, task-specific behavior | Knowledge retrieval, factual Q&A |

**Kết hợp cả hai (phổ biến nhất trong production):**
```
Fine-tuned model (behavior + format) + RAG (knowledge + citations)
= Tốt nhất của cả hai
```

---

## 7. Data Preparation Tips

```typescript
// Chất lượng data > Số lượng data

// ✅ Good training example
const goodExample = {
  messages: [
    { role: "user", content: "Customer reports car stolen from home garage at 2 AM. Police report filed. Third claim this year." },
    { role: "assistant", content: JSON.stringify({
      riskScore: 72,
      redFlags: ["third_claim_this_year", "high_value_late_night"],
      recommendation: "INVESTIGATE",
      reasoning: "Multiple claims pattern combined with late-night timing warrants investigation"
    })},
  ],
};

// ❌ Bad training example — too vague
const badExample = {
  messages: [
    { role: "user", content: "Stolen car" },
    { role: "assistant", content: "This is suspicious" },
  ],
};

// Rules:
// 1. Diverse examples — cover edge cases, not just easy cases
// 2. Consistent format — output format phải IDENTICAL across all examples
// 3. Balance classes — nếu classify, phải có đủ examples của mỗi class
// 4. No PII trong training data — hash/anonymize customer info
// 5. Validation set separate — 10-20% data để measure overfitting
```

---

## Câu hỏi phỏng vấn

**Q: Bạn sẽ chọn fine-tuning hay RAG cho insurance chatbot?**
A: RAG trước vì: (1) Policy documents thay đổi thường xuyên — fine-tuning cần retrain mỗi lần. (2) Cần cite exact source — "theo Điều 5 hợp đồng của bạn...". (3) RAG dễ audit và debug hơn. Fine-tune sau nếu: cần consistent output format, hoặc response quality của RAG không đủ sau nhiều iteration.

**Q: Cần bao nhiêu data để fine-tune?**
A: OpenAI recommend tối thiểu 50 examples để thấy improvement, 100-1000 để kết quả tốt. Với classification task đơn giản, 50-100 chất lượng cao tốt hơn 1000 examples kém chất lượng. Luôn cần validation set để detect overfitting.

**Q: Làm sao biết model đã overfit?**
A: Training loss giảm nhưng validation loss tăng = overfitting. Giải pháp: giảm n_epochs, thêm data, thêm regularization. Với OpenAI, theo dõi training vs validation loss trong job events.
