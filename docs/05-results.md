# 05 · Results

## PDF-to-Excel automation

### Accuracy

Evaluated on a held-out set of **600 real documents** (≈9,400 fields), double-checked by the finance team:

| Metric | Value |
|---|---|
| Field-level accuracy, raw extraction (before validation) | 97.9% |
| **Field-level accuracy, after validation + self-correction** | **99.7%** |
| Document-level accuracy (every field correct) | ~96% |
| Documents flagged for human review | ~4% |
| Errors that reached Excel after optional review | ≈ 0 |

By document type (field-level, final):

| Document type | Accuracy |
|---|---|
| Supplier invoices (native PDF) | 99.9% |
| Supplier invoices (scanned) | 99.6% |
| Customs declarations | 99.5% |
| Export invoices | 99.8% |
| Delivery notes / purchase orders / packing lists | 99.7% |

### Ablation: where the accuracy comes from (scanned invoices)

| Pipeline | Field accuracy |
|---|---|
| Tesseract on raw scan + rules | 91.4% |
| + OpenCV preprocessing (deskew, denoise, binarize) | 95.2% |
| + stamp removal + table-line handling | 96.6% |
| + supplier templates + ML field classifier | 97.8% |
| **+ business validation + self-correction** | **99.6%** |

### Speed

| | Manual | Automated |
|---|---|---|
| 100 documents | ~7 h (~4 min / doc) | **~20 min** (~12 s / doc) |
| + optional review of flagged documents | — | ~10 min |
| Time saved per 100 documents | — | **~6.5 h (−93%)** |

At SCP's volume (roughly **400–500 documents per week**), that frees about **30 hours per week** of staff time. The team moved that time to supplier follow-up and analysis instead of typing.

## Employee assistant (chatbot)

Test set: 200 questions (see [04-rag-chatbot](04-rag-chatbot.md#5-evaluation)).

| Metric | Result |
|---|---|
| Document questions answered correctly | **~91%** (100 / 110) |
| Faithfulness (no unsupported claims) on document answers | ~96% |
| Database questions: correct tool and parameters | ~97% (58 / 60) |
| Database questions: **exact figures when routed correctly** | **100%** |
| Off-topic / adversarial prompts refused | **~97%** (29 / 30) |
| Legitimate work questions wrongly refused | ~2% |
| Median latency (document / database answers) | ~2.3 s / ~1.7 s |
| Average cost per question | **< $0.001** |

**Adoption:** about **60 employees** used it in the first two months, with roughly **300–400 questions per week**. The most common uses were monthly trade figures, invoice status and procedure lookups. The finance team estimated it saved **several hours per week** of "can you check in the Excel file…" requests.

## Infrastructure

| Metric | Value |
|---|---|
| Availability (health-check based) | ~99.8% |
| Security incidents | 0 |
| Monthly AWS cost | ~$230 |
| Database restore drill | Monthly, < 15 min to restore |
