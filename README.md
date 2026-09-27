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

`.env` is gitignored.

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
Analysis.ipynb                     aggregation + figures
Final Pipeline.ipynb               LLM extraction + classification
Data/final_outputs/                91 per-company CSVs (>=1 GBF-mapped statement)
Data/0 statement csvs/             9 companies with no biodiversity statements
Data/reports_manifest.csv          catalogue of all 100 source reports
company_biodiversity_summary.csv   output of Analysis.ipynb cell 0
```

100 companies = 91 analysed + 9 with no extractable statements.

### Source reports are not redistributed

`Data/reports/` (813 MB, 100 PDFs, 18,596 pages) is **not** included. The
reports are the copyright of the respective companies.

`Data/reports_manifest.csv` catalogues all 100 companies, so that the reports can be reassembled from the
original publishers (BSE, NSE, or company websites). Place the PDFs in `Data/reports/` to re-run the extraction pipeline.

`Analysis.ipynb` does **not** need them -- it works entirely from the
committed CSVs, so all reported results are reproducible from this
repository alone.


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

## Licence

- **Code** (notebooks, scripts): MIT -- see `LICENSE`
- **Derived data** (the CSVs under `Data/`, `company_biodiversity_summary.csv`):
  CC BY 4.0 -- see `LICENSE-DATA`
- **Source reports**: copyright of the respective companies, not
  redistributed here

## Citing

See `CITATION.cff`, or use GitHub's "Cite this repository" button.

## References

1. Danaei, M., Gunwal, S., & Nadarajah, S. (2026). *What Companies Say vs.
   What Matters: LLM Analysis of Biodiversity Disclosures in Oil and Gas.*
   EarthArXiv preprint. The primary methodological reference for this work.
2. Centre for Science and Environment (2012). *India's Best Iron and Steel
   Company Gets Average Score; Sector is Rated Poor.*
3. Natural Capital Finance Alliance. *ENCORE: Exploring Natural Capital
   Opportunities, Risks and Exposure.* <https://encorenature.org>
4. Convention on Biological Diversity (2022). *Kunming-Montreal Global
   Biodiversity Framework: The 23 Targets.*
   <https://www.cbd.int/gbf/targets>
