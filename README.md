# Response of Indian Iron and Steel Companies Towards Biodiversity

An LLM-based assessment of corporate biodiversity disclosures and their
alignment with the Kunming–Montreal Global Biodiversity Framework (GBF).

Sagnik Roy (Indian Statistical Institute, Kolkata) ·
Mousami Prasad (Indian Institute of Technology Kanpur)

## Setup

```bash
pip install -r requirements.txt
```

The extraction pipeline needs a Google AI Studio API key
(<https://aistudio.google.com/apikey>). Copy the template and fill it in:

```bash
cp .env.example .env
# then edit .env and set GOOGLE_API_KEY=...
```

`.env` is gitignored. Never commit a key, and never paste one into a
notebook cell.

## Running

Run both notebooks from the **project root** — paths are relative to it.

### 1. `Final Pipeline.ipynb` — extraction and classification

Reads every PDF in `Data/reports/`, and for each one:

1. extracts text page by page (PyMuPDF) and cleans it
2. sends pages over 100 characters to Gemini to pull out
   biodiversity-related statements
3. classifies each statement as a Goal, Commitment or SMART Target,
   and maps it to one or more of the 23 GBF targets
4. writes one CSV per company to `Data/final_outputs/`

Costs one API call per page plus one per extracted statement, so a full
run over 100 reports takes hours. It is resumable: companies that already
have a CSV are skipped, so an interrupted run can simply be restarted.

### 2. `Analysis.ipynb` — aggregation and figures

Reads `Data/final_outputs/*.csv` and writes
`company_biodiversity_summary.csv` (one row per company), then produces
the disclosure-composition and GBF-alignment figures.


## Layout

```
Data/reports/              100 annual / sustainability reports (FY2024-25)
Data/final_outputs/        91 per-company CSVs (>=1 GBF-mapped statement)
Data/0 statement csvs/     9 companies with no biodiversity statements
Plots/                     exported figures
company_biodiversity_summary.csv   output of Analysis.ipynb cell 0
```

100 companies = 91 analysed + 9 with no extractable statements.

## Metrics

**Disclosure composition** — the share of each company's statements that
are Goals, Commitments and SMART Targets. A statement whose five SMART
flags are all true is counted as a SMART Target regardless of the label
the LLM assigned.


**Biodiversity Disclosure Index (BDI)** — for the 11 GBF targets that
ENCORE identifies as material to iron and steel, each target scores
1 if addressed by a Goal, 2 by a Commitment and 3 by a SMART Target
(cumulative), weighted by its materiality $M_i \in \{1..5\}$:

$$\mathrm{BDI} = \frac{\sum_{i \in G} M_i\,(1 \cdot I_G(i) + 2 \cdot I_C(i) + 3 \cdot I_T(i))}{6 \sum_{i \in G} M_i}$$

The denominator (`MAX_INDEX`, = 276) is the maximum attainable score.

## Known gaps

- Section 3.3 of the report (Spearman correlations of BDI against market
  capitalisation and net profit margin) has **no code or data in this
  repository**. It was done ad hoc in R with values as reported on 05/07/2026. 
- Raw LLM responses are not cached, so classifications cannot be
  re-verified exactly. `temperature=0` does not guarantee stability
  across server-side model revisions.
- `google-generativeai` is end-of-life; migrate to `google-genai`.
