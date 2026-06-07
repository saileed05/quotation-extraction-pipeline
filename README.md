# Quotation Data Extraction Pipeline v3.0.0

A production document intelligence system built for a real B2B business client. Processes 680+ mixed-format quotation files (PDF, DOCX, XLSX, images) and extracts structured data using a hybrid regex + LLM approach — replacing a fully manual workflow.

---

## What it does

Sales teams receive quotations in dozens of inconsistent formats from 165+ clients. This pipeline reads every document, extracts key fields, validates them, and outputs a clean structured Excel report and SQLite database — automatically.

**Fields extracted per document:**
- Quotation number
- Client name (with fuzzy alias resolution)
- Date
- Product type
- Area, rate, and amount (with cross-validation)
- Salesperson

---

## Architecture — 7-stage pipeline

```
Input (ZIP of mixed files)
        │
        ▼
Stage 1 — Document Extraction
         PDF / DOCX / XLSX / Image → raw text
         OCR fallback for scanned documents (Tesseract)
        │
        ▼
Stage 2 — Regex Extraction
         Fast pattern matching for structured fields
         Filename parsing as additional signal
        │
        ▼
Stage 3 — LLM Extraction (Groq LLaMA-3.3-70B)
         Handles unstructured, inconsistent formats
         Structured JSON output prompting
         Token bucket rate limiter (25 RPM)
         MD5-based response caching
        │
        ▼
Stage 4 — Confidence Scoring
         Per-field confidence score (0.0 – 1.0)
         Source tracking: regex / llm / filename / table
         Records below 60% threshold flagged for review
        │
        ▼
Stage 5 — Validation & Deduplication
         Cross-validation: area × rate vs amount (±6% tolerance)
         JSONL deduplication with MD5 hash guards
         False positive detection
        │
        ▼
Stage 6 — Correction Learning (SQLite)
         User corrections stored and applied on future runs
         Continuous improvement across pipeline runs
        │
        ▼
Stage 7 — Report Generation
         Structured Excel output (.xlsx)
         SQLite database for querying
         Stage-level logs for root-cause tracing
```

---

## Key engineering features

| Feature | Detail |
|---|---|
| **Multi-format support** | PDF, DOCX, XLSX, JPG, PNG — single unified extractor |
| **OCR fallback** | Tesseract for scanned/image-based documents |
| **Fuzzy client matching** | 165 real clients with alias dictionaries |
| **Confidence scoring** | Per-field score with auto-flagging below threshold |
| **Rate limiting** | Thread-safe token bucket (25 RPM) for Groq API |
| **LLM caching** | MD5-based response cache — avoids redundant API calls |
| **Mid-run resume** | Checkpoint system — pipeline resumes from last processed file |
| **Deduplication** | JSONL hash guards prevent duplicate records |
| **Cross-validation** | area × rate vs amount within 6% tolerance |
| **Interactive review UI** | ipywidgets-based UI for correcting flagged records |

---

## Tech stack

- **LLM:** Groq API (LLaMA-3.3-70B)
- **Document parsing:** pdfplumber, python-docx, Pillow, Tesseract OCR
- **Data:** pandas, SQLite, OpenPyXL
- **Runtime:** Google Colab (production), Python 3.10+

---

## Setup

### 1. Clone the repo
```bash
git clone https://github.com/saileed05/quotation-extraction-pipeline.git
cd quotation-extraction-pipeline
```

### 2. Install dependencies
```bash
pip install -r requirements.txt
```

### 3. Install Tesseract OCR
```bash
# Ubuntu/Debian
sudo apt-get install tesseract-ocr

# Mac
brew install tesseract
```

### 4. Set your Groq API key
In the notebook, update the config cell:
```python
GROQ_API_KEY = "your_groq_api_key_here"
```
Get a free key at [console.groq.com](https://console.groq.com)

### 5. Run in Google Colab
Upload the notebook to Colab, mount your Google Drive, and point `config.input_zip` to your ZIP file of documents.

---

## Usage

```python
# Process a ZIP of documents
results = pipeline.run_from_zip(Path("your_documents.zip"))

# Generate Excel report
report_gen.generate(results)

# Process new inbox files (no duplicates)
inbox_processor.process_inbox()

# Reset and reprocess everything
# Run the RESET cell in the notebook
```

---

## Output

- `extractions.jsonl` — raw extraction results with confidence scores
- `quotation_report_[timestamp].xlsx` — clean Excel report
- `corrections.db` — SQLite database of corrections and all records
- `pipeline.log` — stage-level execution log

---

## Note on data

All client names, quotation data, and business files used during development are confidential and not included in this repository. The pipeline is designed to work with your own document corpus.

---

## Author

**Sailee Desai**
[Portfolio](https://sailee-desai-portfolio.vercel.app) · [LinkedIn](https://linkedin.com/in/sailee-desai) · [GitHub](https://github.com/saileed05)
