# Phase 10 — Multimodal & OCR

> Insurance documents là PDF, scanned images, forms, tables, handwritten notes.
> Text chunking đơn giản không đủ — bạn cần handle visual content, tables,
> structured forms, và scanned documents. Đây là core skill cho InsurTech.

---

## 1. Tại sao Multimodal quan trọng trong Insurance

```
Insurance documents bao gồm:
  - Policy documents (PDF, text-based)           → pdf-parse đủ
  - Claim forms (PDF với tables, checkboxes)      → cần table extraction
  - Medical records (scan, handwriting)           → OCR cần thiết
  - Police reports (scan hoặc typed)             → OCR / vision model
  - Damage photos (JPEG, PNG)                    → Vision model
  - Invoices, receipts (PDF hoặc image)          → structured extraction
  - Driver's license, ID cards (image)           → OCR + PII handling
```

---

## 2. PDF Processing — Strategy Selection

```
Loại PDF                         Strategy tốt nhất
─────────────────────────────────────────────────────
Text-based PDF (digital)         pdf-parse / pdfjs
PDF với tables                   AWS Textract / Google DocAI
Scanned PDF (image-based)        OCR service + LLM vision
Mixed (text + scan + tables)     Multi-step pipeline
Form PDF (fillable)              pdf-parse + form field extract
Handwritten notes                LLM vision (GPT-4o / Claude)
```

---

## 3. pdf-parse — Text-based PDF (bạn đã biết)

```typescript
import pdf from "pdf-parse";
import fs from "fs";

// Basic text extraction
const buffer = fs.readFileSync("policy.pdf");
const data = await pdf(buffer);

console.log(data.text);       // extracted text
console.log(data.numpages);   // page count
console.log(data.info);       // metadata (author, created date, etc.)

// Limitation: mất formatting, tables trở thành text rối
// "Coverage | Amount" → "Coverage Amount" hoặc tệ hơn
```

### Xử lý tốt hơn với page-by-page

```typescript
// Custom render để preserve structure
const options = {
  pagerender: async (pageData: any) => {
    const textContent = await pageData.getTextContent();

    // Nhóm text items theo Y position để detect rows
    const items = textContent.items as any[];
    const rows = new Map<number, string[]>();

    items.forEach((item) => {
      const y = Math.round(item.transform[5]);  // Y coordinate
      if (!rows.has(y)) rows.set(y, []);
      rows.get(y)!.push(item.str);
    });

    // Sort by Y (top to bottom), join each row
    const sortedRows = [...rows.entries()]
      .sort(([a], [b]) => b - a)  // descending Y = top to bottom
      .map(([, texts]) => texts.join(" "));

    return sortedRows.join("\n");
  },
};

const data = await pdf(buffer, options);
```

---

## 4. AWS Textract — Tables và Forms

AWS Textract là managed OCR service tốt nhất cho documents có tables và forms.

### Setup

```bash
npm install @aws-sdk/client-textract
```

```typescript
import {
  TextractClient,
  AnalyzeDocumentCommand,
  DetectDocumentTextCommand,
} from "@aws-sdk/client-textract";
import fs from "fs";

const client = new TextractClient({ region: "us-east-1" });
```

### Extract text (scanned PDF → text)

```typescript
async function extractTextFromScannedPDF(buffer: Buffer): Promise<string> {
  const command = new DetectDocumentTextCommand({
    Document: { Bytes: buffer },
  });

  const response = await client.send(command);

  // Extract text blocks in reading order
  const textBlocks = response.Blocks?.filter((b) => b.BlockType === "LINE") ?? [];
  return textBlocks.map((b) => b.Text ?? "").join("\n");
}
```

### Extract tables — Insurance forms thường có tables

```typescript
interface TableData {
  headers: string[];
  rows: string[][];
}

async function extractTablesFromDocument(buffer: Buffer): Promise<TableData[]> {
  const command = new AnalyzeDocumentCommand({
    Document: { Bytes: buffer },
    FeatureTypes: ["TABLES", "FORMS"],  // extract tables AND form key-values
  });

  const response = await client.send(command);
  const blocks = response.Blocks ?? [];

  // Build block map for relationship traversal
  const blockMap = new Map(blocks.map((b) => [b.Id, b]));

  // Extract tables
  const tables: TableData[] = [];
  const tableBlocks = blocks.filter((b) => b.BlockType === "TABLE");

  for (const table of tableBlocks) {
    const cells = new Map<string, string>();  // "row,col" → text
    let maxRow = 0;
    let maxCol = 0;

    // Traverse table cells
    table.Relationships?.forEach((rel) => {
      if (rel.Type === "CHILD") {
        rel.Ids?.forEach((cellId) => {
          const cell = blockMap.get(cellId);
          if (cell?.BlockType === "CELL") {
            const row = cell.RowIndex ?? 0;
            const col = cell.ColumnIndex ?? 0;
            maxRow = Math.max(maxRow, row);
            maxCol = Math.max(maxCol, col);

            // Get cell text
            const cellText = cell.Relationships?.flatMap((r) =>
              r.Type === "CHILD"
                ? r.Ids?.map((id) => blockMap.get(id)?.Text ?? "") ?? []
                : []
            ).join(" ") ?? "";

            cells.set(`${row},${col}`, cellText);
          }
        });
      }
    });

    // Convert to 2D array
    const grid: string[][] = [];
    for (let r = 1; r <= maxRow; r++) {
      const row: string[] = [];
      for (let c = 1; c <= maxCol; c++) {
        row.push(cells.get(`${r},${c}`) ?? "");
      }
      grid.push(row);
    }

    tables.push({
      headers: grid[0] ?? [],
      rows: grid.slice(1),
    });
  }

  return tables;
}

// Ví dụ output cho insurance claim table:
// {
//   headers: ["Coverage Type", "Limit", "Deductible", "Premium"],
//   rows: [
//     ["Bodily Injury", "$100,000", "$500", "$450"],
//     ["Property Damage", "$50,000", "$500", "$220"],
//     ["Comprehensive", "$ACV", "$250", "$180"],
//   ]
// }
```

### Extract Key-Value pairs (Form fields)

```typescript
async function extractFormFields(buffer: Buffer): Promise<Record<string, string>> {
  const command = new AnalyzeDocumentCommand({
    Document: { Bytes: buffer },
    FeatureTypes: ["FORMS"],
  });

  const response = await client.send(command);
  const blocks = response.Blocks ?? [];
  const blockMap = new Map(blocks.map((b) => [b.Id, b]));

  const fields: Record<string, string> = {};

  // KEY_VALUE_SET blocks contain form key-value pairs
  blocks
    .filter((b) => b.BlockType === "KEY_VALUE_SET" && b.EntityTypes?.includes("KEY"))
    .forEach((keyBlock) => {
      // Get key text
      const keyText = keyBlock.Relationships?.flatMap((r) =>
        r.Type === "CHILD"
          ? r.Ids?.map((id) => blockMap.get(id)?.Text ?? "") ?? []
          : []
      ).join(" ") ?? "";

      // Get value block
      const valueBlockId = keyBlock.Relationships?.find(
        (r) => r.Type === "VALUE"
      )?.Ids?.[0];

      if (valueBlockId) {
        const valueBlock = blockMap.get(valueBlockId);
        const valueText = valueBlock?.Relationships?.flatMap((r) =>
          r.Type === "CHILD"
            ? r.Ids?.map((id) => blockMap.get(id)?.Text ?? "") ?? []
            : []
        ).join(" ") ?? "";

        fields[keyText.trim()] = valueText.trim();
      }
    });

  return fields;
  // {
  //   "Claimant Name:": "Nguyen Van A",
  //   "Date of Incident:": "01/15/2024",
  //   "Policy Number:": "POL-2024-001234",
  //   "Description of Loss:": "Car rear-ended at traffic light",
  // }
}
```

---

## 5. Google Document AI — Alternative to Textract

```bash
npm install @google-cloud/documentai
```

```typescript
import { DocumentProcessorServiceClient } from "@google-cloud/documentai";

const client = new DocumentProcessorServiceClient();

// Google có pre-built processors cho insurance documents
const processors = {
  generalOCR: "projects/{project}/locations/us/processors/{id}",
  formParser: "projects/{project}/locations/us/processors/{id}",
  invoiceParser: "projects/{project}/locations/us/processors/{id}",
  identityDocParser: "projects/{project}/locations/us/processors/{id}",
};

async function processWithGoogleDocAI(buffer: Buffer, processorName: string) {
  const [result] = await client.processDocument({
    name: processorName,
    rawDocument: {
      content: buffer.toString("base64"),
      mimeType: "application/pdf",
    },
  });

  const document = result.document!;

  // Entities (structured data extracted by pre-built processors)
  const entities = document.entities?.map((e) => ({
    type: e.type,
    mentionText: e.mentionText,
    confidence: e.confidence,
    normalizedValue: e.normalizedValue?.text,
  }));

  return { text: document.text, entities };
}
```

---

## 6. Azure Document Intelligence — Tốt cho Microsoft stack

```bash
npm install @azure-rest/ai-document-intelligence @azure/core-auth
```

```typescript
import DocumentIntelligence from "@azure-rest/ai-document-intelligence";

const client = DocumentIntelligence(
  process.env.AZURE_DOC_INTELLIGENCE_ENDPOINT!,
  { key: process.env.AZURE_DOC_INTELLIGENCE_KEY! }
);

// Prebuilt models:
// prebuilt-read       — text extraction
// prebuilt-layout     — text + tables + structure
// prebuilt-document   — general document with key-values
// prebuilt-invoice    — invoices (useful cho claim receipts)
// prebuilt-idDocument — ID cards, driver's license
// prebuilt-healthInsuranceCard — health insurance specific!

async function extractInsuranceCard(imageBuffer: Buffer) {
  const poller = await client
    .path("/documentModels/{modelId}:analyze", "prebuilt-healthInsuranceCard")
    .post({
      contentType: "application/octet-stream",
      body: imageBuffer,
    });

  const result = await poller.json();

  // Structured fields specific to health insurance cards
  const card = result.analyzeResult?.documents?.[0]?.fields;
  return {
    memberName: card?.MemberName?.content,
    memberId: card?.MemberId?.content,
    groupNumber: card?.GroupNumber?.content,
    policyNumber: card?.PolicyNumber?.content,
    deductible: card?.Deductible?.content,
    copay: card?.Copay?.content,
    insurer: card?.Insurer?.content,
  };
}
```

---

## 7. LLM Vision — Best cho Complex Documents

Khi OCR không đủ — handwriting, complex layouts, damage photos:

```typescript
import Anthropic from "@anthropic-ai/sdk";
import fs from "fs";

const client = new Anthropic();

// Convert PDF page to image trước (dùng pdf2pic hoặc sharp)
import { fromBuffer } from "pdf2pic";

async function pdfToImages(pdfBuffer: Buffer): Promise<Buffer[]> {
  const convert = fromBuffer(pdfBuffer, {
    density: 200,        // DPI — cao hơn = chất lượng hơn
    format: "jpeg",
    width: 2480,
    height: 3508,
  });

  const pageCount = 5;  // hoặc detect từ pdf-parse
  const images: Buffer[] = [];

  for (let i = 1; i <= pageCount; i++) {
    const result = await convert(i, { responseType: "buffer" });
    images.push(result.buffer as Buffer);
  }

  return images;
}

// Extract từ complex document với LLM vision
async function extractWithVision(imageBuffer: Buffer): Promise<Record<string, any>> {
  const base64 = imageBuffer.toString("base64");

  const response = await client.messages.create({
    model: "claude-opus-4-8",  // best vision model
    max_tokens: 2048,
    messages: [{
      role: "user",
      content: [
        {
          type: "image",
          source: {
            type: "base64",
            media_type: "image/jpeg",
            data: base64,
          },
        },
        {
          type: "text",
          text: `Extract ALL information from this insurance document.
Return as JSON with these fields:
- documentType: type of document
- claimantInfo: { name, dob, address, phone, email, policyNumber }
- incidentDetails: { date, time, location, description, witnesses }
- damages: [{ item, description, estimatedValue }]
- signatures: [{ signerName, date, role }]
- tables: [{ title, headers, rows }]
- handwrittenNotes: [{ location, content }]
- missingOrIllegible: [{ field, reason }]

For any field not found, use null.
Preserve exact monetary values, dates, and ID numbers.`,
        },
      ],
    }],
  });

  return JSON.parse(response.content[0].text);
}

// Damage photo analysis
async function analyzeDamagePhoto(imageBuffer: Buffer, claimType: string) {
  const base64 = imageBuffer.toString("base64");

  const response = await client.messages.create({
    model: "claude-opus-4-8",
    max_tokens: 1024,
    messages: [{
      role: "user",
      content: [
        {
          type: "image",
          source: { type: "base64", media_type: "image/jpeg", data: base64 },
        },
        {
          type: "text",
          text: `Analyze this ${claimType} damage photo for insurance purposes.
Return JSON:
{
  "damageVisible": boolean,
  "damageDescription": string,
  "affectedAreas": string[],
  "estimatedSeverity": "minor" | "moderate" | "severe" | "total_loss",
  "consistentWithClaim": boolean | null,
  "suspiciousIndicators": string[],
  "recommendedNextSteps": string[]
}`,
        },
      ],
    }],
  });

  return JSON.parse(response.content[0].text);
}
```

---

## 8. Full Document Processing Pipeline (Insurance)

```typescript
// NestJS Service — orchestrate toàn bộ pipeline
@Injectable()
export class InsuranceDocumentProcessor {
  constructor(
    private textractService: TextractService,
    private llmService: LLMService,
    private embeddingService: EmbeddingService,
    private vectorStore: VectorStore,
  ) {}

  async processDocument(
    buffer: Buffer,
    mimeType: string,
    metadata: DocumentMetadata
  ): Promise<ProcessedDocument> {

    // Step 1: Detect document type
    const docType = await this.detectDocumentType(buffer, mimeType);

    // Step 2: Extract content based on type
    let extractedContent: DocumentContent;

    if (mimeType === "application/pdf") {
      // Try text extraction first (free, fast)
      const textResult = await this.extractPDFText(buffer);

      if (textResult.isTextBased && !textResult.hasTables) {
        // Simple text PDF
        extractedContent = { text: textResult.text, tables: [], forms: {} };
      } else if (textResult.hasTables || textResult.hasForms) {
        // Use Textract for tables/forms
        const [tables, forms] = await Promise.all([
          this.textractService.extractTables(buffer),
          this.textractService.extractFormFields(buffer),
        ]);
        extractedContent = { text: textResult.text, tables, forms };
      } else {
        // Scanned PDF — convert to images and use vision
        const images = await this.convertPDFToImages(buffer);
        extractedContent = await this.extractWithVision(images);
      }
    } else if (mimeType.startsWith("image/")) {
      // Direct image — vision model
      extractedContent = await this.extractWithVision([buffer]);
    } else {
      throw new Error(`Unsupported document type: ${mimeType}`);
    }

    // Step 3: Structure the data
    const structured = await this.structureWithLLM(extractedContent, docType);

    // Step 4: Index for RAG
    await this.indexForRAG(extractedContent.text, metadata);

    // Step 5: Save to database
    await this.saveToDatabase(structured, metadata);

    return { structured, extractedContent, metadata };
  }

  private async detectDocumentType(buffer: Buffer, mimeType: string): Promise<string> {
    // Quick heuristic first
    if (mimeType === "image/jpeg" || mimeType === "image/png") {
      // Could be ID card, damage photo, or scanned document
      // Use vision to detect
    }

    // Use LLM to classify from first page text
    const text = await this.extractPDFText(buffer);
    const firstPage = text.text.slice(0, 2000);

    const docType = await this.llmService.complete(
      [{ role: "user", content: `Classify this insurance document type from its content. Return one of: policy_document, claim_form, medical_record, police_report, invoice, id_document, damage_photo, other\n\nContent:\n${firstPage}` }],
      { temperature: 0 }
    );

    return docType.trim();
  }

  private async indexForRAG(text: string, metadata: DocumentMetadata) {
    const splitter = new RecursiveCharacterTextSplitter({
      chunkSize: 1000,
      chunkOverlap: 200,
    });
    const chunks = await splitter.splitText(text);
    const embeddings = await this.embeddingService.embedBatch(chunks);

    await this.vectorStore.upsert(
      chunks.map((content, i) => ({
        id: `${metadata.documentId}_${i}`,
        content,
        embedding: embeddings[i],
        metadata,
      }))
    );
  }
}
```

---

## 9. Table → Markdown → Embed (Best Practice)

Tables mất semantic khi convert to plain text. Tốt hơn: convert to Markdown trước khi embed.

```typescript
function tableToMarkdown(table: TableData): string {
  const { headers, rows } = table;

  // Header row
  const headerRow = `| ${headers.join(" | ")} |`;
  // Separator
  const separator = `| ${headers.map(() => "---").join(" | ")} |`;
  // Data rows
  const dataRows = rows.map((row) => `| ${row.join(" | ")} |`).join("\n");

  return [headerRow, separator, dataRows].join("\n");
}

// Ví dụ output:
// | Coverage Type | Limit | Deductible |
// | --- | --- | --- |
// | Bodily Injury | $100,000 | $500 |
// | Property Damage | $50,000 | $500 |

// Embedding Markdown table → semantic search hoạt động tốt hơn
// vì LLM hiểu Markdown table format
```

---

## Câu hỏi phỏng vấn

**Q: Bạn sẽ xử lý scanned insurance claim form như thế nào?**
A: Pipeline: (1) Detect nếu PDF là scanned (dùng pdf-parse — nếu text rất ít thì là scanned). (2) Convert PDF pages thành high-res JPEG (200 DPI). (3) Gửi cho AWS Textract AnalyzeDocument với FORMS feature — tự động extract key-value pairs. (4) Với handwritten sections, dùng LLM vision (Claude Opus). (5) Validate extracted data với Zod schema và flag missing required fields. (6) Lưu structured data vào DB và index text content vào vector store cho RAG.

**Q: Textract vs Google DocAI vs Azure Document Intelligence — chọn cái nào?**
A: Nếu infrastructure đã trên AWS → Textract (native integration, IAM, S3). Nếu cần pre-built model cụ thể (health insurance card) → Azure Document Intelligence (có prebuilt-healthInsuranceCard model). Nếu cần tốt nhất cho general documents với nhiều languages → Google DocAI. Nếu muốn zero infra → GPT-4o hoặc Claude vision trực tiếp (đắt hơn nhưng đơn giản nhất).

**Q: Làm sao handle documents có cả text lẫn images?**
A: Multi-pass: (1) pdf-parse extract text layer. (2) Detect embedded images trong PDF (kiểm tra có images không bằng pdf-parse info). (3) Render full page as image, gửi cho vision model để extract từ images và verify/supplement text extraction. (4) Merge kết quả: text từ pdf-parse + image content từ vision model.
