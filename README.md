# Gemini vs GPT-4o: Financial Document Extraction Benchmarking

A structured document intelligence pipeline that extracts key financial fields 
from annual reports using two LLMs — Gemini 2.5 Flash and GPT-4o Mini — and 
scores extraction accuracy against ground truth.

Built as a direct extension of my RAG Financial Analyzer, reframed from 
freeform Q&A to structured field extraction with a formal eval harness.

---

## What This Does

1. Ingests a PDF annual report (Berkshire Hathaway 2023) using pdfplumber
2. Extracts structured fields via a prompt-engineered extraction template
3. Validates output against a Pydantic schema
4. Scores both models against manually verified ground truth
5. Produces a side-by-side comparison table

---

## Results

| Field | Gemini 2.5 Flash | GPT-4o Mini | Winner |
|---|---|---|---|
| company_name | 0.80 | 0.80 | Tie |
| fiscal_year | 1.00 | 1.00 | Tie |
| net_income | 1.00 | 1.00 | Tie |
| total_assets | 0.00 | 1.00 | GPT-4o |
| ceo_name | 0.00 | 0.00 | Tie |
| primary_business_segments | 0.00 | 0.00 | Tie |
| **overall** | **0.56** | **0.76** | **GPT-4o** |

**GPT-4o Mini outperformed Gemini 2.5 Flash overall (0.76 vs 0.56)**, 
primarily due to better total_assets extraction from the same context window.

Zero scores on ceo_name and primary_business_segments are a context window 
issue — both fields appear outside the evaluated document slice — not a model 
failure. This is a known limitation documented for future work.

---

## Key Design Decisions

**Deterministic extraction over freeform Q&A**
The pipeline extracts specific fields into a typed schema rather than answering 
open-ended questions. This is closer to production document AI use cases in 
financial services.

**Pydantic validation as a quality gate**
Every LLM response is validated against a typed schema before scoring. Malformed 
JSON or missing required fields raise a ValidationError rather than silently 
passing bad data downstream.

**Fuzzy scoring over exact match**
String fields use substring matching to avoid penalizing semantically correct 
extractions that differ in formatting (e.g. "Berkshire Hathaway" vs 
"Berkshire Hathaway Inc.").

**Ground truth is manually verified**
Scores are computed against human-verified field values, not against one model's 
output. This is the correct eval pattern for document extraction tasks.

---

## Stack

- Python, pdfplumber, Pydantic
- Google Gemini 2.5 Flash (`google-generativeai`)
- OpenAI GPT-4o Mini
- pandas for comparison reporting

---

## Future Work

- Expand context window to cover full document for complete field coverage
- Add more document types: 10-K filings, loan applications, contracts
- Implement chunked extraction with field-level context targeting
- Add confidence scores per extracted field

---

## Related Projects

- [RAG Financial Analyzer](https://github.com/twinklebhakri/rag-financial-analyzer) — 
  freeform Q&A over financial reports using FAISS + sentence-transformers + OpenAI
- [Autonomous EDA Agent](https://github.com/twinklebhakri/eda-agent) — 
  pre-modeling data audit agent with structured eval harness
