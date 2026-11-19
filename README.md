# Pittsburgh Capital Budget — PDF → Structured Data (with OCR)

A Python pipeline that turns **25 years of the City of Pittsburgh's Capital Improvement Program budgets (2001–2025)** — published only as PDFs, in five incompatible layouts, some of them *scanned images* — into clean, analysis-ready CSVs.

**2,217 capital projects** extracted across 25 documents, one CSV per budget year, with a consistent 20-field schema.

> The early budgets (2001–2004) are scanned paper documents with no embedded text. The pipeline detects these and runs **Tesseract OCR** to recover the text before parsing — so the same structured schema comes out whether the source was born-digital or a scan of a printout.

---

## Why this was hard

Pittsburgh never published this data in a usable form, and the document format changed repeatedly over 25 years. The notebook handles **five distinct format families** with a single routing pipeline:

| Format | Years | Challenge |
|--------|-------|-----------|
| **A** | 2001–2004 | **Scanned images — no text layer.** Re-OCR'd with Tesseract, then parsed from reconstructed word geometry. |
| **B** | 2005–2010 | Capital-project summary tables. |
| **C** | 2011 | Labelled detail sheets (one-off layout). |
| **D** | 2012–2013 | Labelled detail sheets, revised layout. |
| **E** | 2014–2025 | Modern multi-year funding tables. |

Each PDF is classified page-by-page (`project` vs. everything else), routed to the correct parser, and reconciled against the printed totals in a diagnostics pass.

## How it works

```
classify pages  →  detect format family per PDF  →  [OCR if scanned]  →  route to parser  →  write CSV  →  diagnostics
```

- **`pdfplumber`** — word-level geometry (x/y position, font size) to reconstruct tables that have no real table structure in the PDF.
- **`PyMuPDF` (`fitz`)** — fast plain-text pass for page classification.
- **`pytesseract` (Tesseract OCR)** — recovers text from the scanned 2001–2004 sheets.
- **`pandas`** — assembles records and writes one CSV per budget year.
- **`re`** — field extraction from the reconstructed text.

## Output schema

Each row is one capital project:

```
cip_year, project_type, source_page, department, project_name, project_id,
address_location, start_year, end_year, project_description, project_justification,
previous_appropriations, project_total, year_2025 … year_2031
```

All monetary values are normalized to thousands of dollars. Per-year funding is split into one `year_YYYY` column per planning year.

## What's in this repo

```
pittsburgh_cip_to_csv.ipynb   the pipeline (run top-to-bottom)
CSV/                          all 25 output files, 2001.csv … 2025.csv (the full result)
PDF/
  2025.pdf                    sample input — modern layout (Format E)
  2004.pdf                    sample input — scanned document (Format A, OCR path)
```

The full 2001–2025 PDF set is ~367 MB and is not committed. The two sample PDFs let you run the notebook end-to-end and exercise both the digital and the OCR branch. The remaining source PDFs come from the City of Pittsburgh's published capital budget archives.

## Running it

```bash
pip install pandas pdfplumber pymupdf pytesseract
# Tesseract itself (for the OCR path):  brew install tesseract   # macOS
jupyter notebook pittsburgh_cip_to_csv.ipynb
```

Run the cells top-to-bottom from inside the `Pittsburgh/` folder. Each PDF in `PDF/` is parsed and written to `CSV/<year>.csv`.

---

*Part of a larger research effort converting U.S. municipal Capital Improvement Program budgets into a standardized dataset. Pittsburgh was the hardest city: the widest span of years, the most format changes, and the only one requiring OCR of scanned source documents.*
