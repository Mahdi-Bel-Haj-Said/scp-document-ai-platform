# 06 · Challenges, Trade-offs & Lessons

## Challenges

### 1. Stamps and signatures on top of totals
**Problem:** blue and red company stamps were often placed right over the totals, and Tesseract read them as garbage digits.
**Solution:** HSV colour masking to isolate coloured ink, then inpainting before OCR. When the total was still unreadable, the validation equation (`HT + VAT + stamp = TTC`) **reconstructed the missing value** from the other fields and flagged it for review.

### 2. Tunisian number formats
**Problem:** `1.234,567`, `1 234,567` and `1234.567` all appear, sometimes in the same document. TND has 3 decimals and EUR/USD have 2. One wrong guess makes an amount off by a factor of 1,000.
**Solution:** a currency-aware amount parser that uses the document's currency, the number of decimals and the totals equation to choose the only interpretation that balances.

### 3. Hundreds of supplier layouts
**Problem:** a template per supplier doesn't scale, and generic rules miss unusual layouts.
**Solution:** three strategies (templates for the top 30 suppliers, generic anchor rules, and an ML field classifier for the long tail), with corrections from the review screen fed back as training data.

### 4. Exact figures from a chatbot
**Problem:** early prototypes that put Excel rows into the vector store gave *plausible but wrong* totals.
**Solution:** separate the problems. RAG handles text. Parameterized SQL tools handle numbers. An output check rejects any number that doesn't come from a tool result.

### 5. Keeping the assistant on-topic without annoying users
**Problem:** a strict filter refused legitimate questions ("what's the VAT rate on imported raw materials?"), and a loose one let anything through.
**Solution:** a two-stage topic check (fast embedding similarity, then an LLM classifier only when uncertain), tuned on real questions until false refusals were ~2%.

### 6. Security of financial data in the cloud
**Problem:** management was worried about putting financial documents on AWS.
**Solution:** private subnets, no SSH, instance roles instead of keys, encryption at rest and in transit, a VPC-endpoint-only bucket policy, audited admin access through SSM, and a documented architecture review with the IT manager before go-live.

## Trade-offs

| Decision | Alternative | Why |
|---|---|---|
| Tesseract (self-hosted) | AWS Textract / cloud OCR APIs | Data stays inside the VPC, no per-page cost at this volume, and with preprocessing + validation the accuracy target was reached anyway |
| PostgreSQL on EC2 | Amazon RDS | Budget. Backups, snapshots and restore drills covered the main risks. **RDS would be my first upgrade** |
| Predefined SQL tools | Free text-to-SQL | Safety and exactness matter more than flexibility. New tools are cheap to add when a new question type appears |
| `gpt-4o-mini` | Larger models | Answers are grounded in tools and documents, so a small model was accurate enough, ~10× cheaper and faster |
| Single-AZ deployment | Multi-AZ with a load balancer | Internal tool with business-hours usage. The ~99.8% availability met the requirement at a fraction of the cost |

## What I'd do next

- Migrate the database to **Amazon RDS (Multi-AZ)** and the frontend to **S3 + CloudFront**.
- Try **layout-aware document models** (e.g. LayoutLMv3 / Donut) fine-tuned on the 1,500+ corrected documents, to reduce the need for supplier templates.
- **Email ingestion:** process invoices automatically from a dedicated mailbox.
- **ERP integration** through an API instead of Excel, once the ERP supports it.
- Add a **re-ranker** (cross-encoder) to RAG retrieval, and more SQL tools based on the audit log of unanswered questions.

## Lessons

1. **Business rules are the best accuracy booster.** Totals and VAT checks turned a 97.9% extractor into a 99.7% system.
2. **Keep humans in the loop where it's cheap.** Reviewing 4% of documents is fast and builds trust in the remaining 96%.
3. **Don't ask an LLM to do arithmetic over data.** Give it tools.
4. **Fit the existing workflow.** Filling the *same* Excel templates people already used made adoption immediate.
