# Predicting Single-Bid Public Procurement Contracts in Bulgaria

> **🚧 Work in progress.** The data foundation (exploration, cleaning, problem
> framing) is complete; modelling and evaluation are in progress. Section 8 of
> `report.ipynb` (Results and conclusions) will be filled in as the project
> develops.

Can a contract's characteristics predict whether it will attract only a
**single bid** - a recognised competition red-flag indicator (not a measure of
corruption)? This project builds a classifier on Bulgarian public procurement
data and examines which characteristics drive the pattern.

The unit of analysis is the **contract**, not the company.

> **Start here:** `report.ipynb` (root) is the project write-up - problem,
> framing, significance, scope and (by the end) results and conclusions. The
> numbered notebooks in `notebooks/` carry the step-by-step analysis.

## What this project is - and isn't

The model points at contracts where competition looks weak and says "worth a look
here." It does not, and cannot, say anything illegal happened. A single bid is a
competition *outcome*, not proof of wrongdoing - many single-bid contracts are
completely clean.

This framing drives two choices later in the project:

- **Precision over bare accuracy.** Whoever uses these flags - an auditor, a
  journalist - cannot investigate tens of thousands of contracts, so the flags
  must be meaningful. The models are judged on precision / recall / F1 / ROC-AUC,
  not accuracy alone.
- **No leakage from post-award information.** Only characteristics known around
  announcement time are used. The contractor's identity is known only *after* the
  award, so it is excluded; aggregate per-authority columns (total spend, contract
  and supplier counts, averages) are excluded too, since they summarise outcomes
  that overlap with the target.

**Main limitation, stated up front.** One feature - `procedure` - is partly
circular with the target. Some procedure types, above all "Пряко / без обявление"
(direct award), are *defined* as effectively single-source, so for those
contracts a single bid is structural rather than behavioural. This is the
project's headline limitation: it is measured in the exploration, investigated by
modelling with and without that procedure, and written up in Limitations. The EC
methodology gives cover here - the Commission also keeps direct awards out of its
single-bidder indicator, for the same reason.

## Data

The project uses Bulgarian public procurement data from **SIGMA** - a public
transparency platform for Bulgarian public procurement, hosted under the domain
of the **Ministry of Innovation and Digital Transformation** (МИДТ,
[midt.bg](https://www.midt.bg/)):

- `https://sigma.midt.bg/contracts`
- `https://sigma.midt.bg/authorities`
- `https://sigma.midt.bg/companies`

SIGMA consolidates open data from the official Bulgarian Public Procurement
Register, **АОП / ЦАИС ЕОП**, made available through `storage.eop.bg`.

The analysis combines three CSV exports:

- **Contracts** - awarded public procurement contracts and lots. This is the main
  dataset and the unit of analysis for the model.
- **Authorities** - contracting authorities / institutions that awarded at least
  one procurement contract.
- **Companies** - suppliers / contractors that won at least one procurement
  contract.

Although authority-level and company-level information is used as additional
context, the prediction task is defined at the **contract / lot level**, not at
the company level. The model predicts whether a given contract attracted only a
**single bid**, using contract characteristics and related metadata.

### Obtaining the data

The CSV files are not tracked in this repository because of their size. There are
two ways to get them.

**Option A - the exact snapshot used here (recommended for reproducibility).**
The three CSVs analysed in this project are archived as
[`sigma-data-16-june-2026.zip`](../../releases/latest) under the repository's
GitHub Releases. Download it and unzip into `data/raw/` so the files land as:

```text
data/raw/
  sigma-contracts.csv
  sigma-authorities.csv
  sigma-companies.csv
```

**Option B - a fresh download from SIGMA.** Each list on SIGMA has a direct CSV
export (the **"Изтегли CSV" / "Download CSV"** link points to a `.csv` URL):

- `https://sigma.midt.bg/contracts.csv`
- `https://sigma.midt.bg/authorities.csv`
- `https://sigma.midt.bg/companies.csv`

Save them under `data/raw/` with the same names as above. SIGMA updates daily, so
a fresh download will be larger than the analysed snapshot - the analysis still
runs, but exact figures will shift slightly from those reported here.

### Snapshot used in this project

All figures in the notebooks are computed from a fixed snapshot downloaded on
**June 16, 2026**. To confirm any copy (the Release zip *or* a fresh download)
matches it, check the row counts and MD5 checksums:

| File                    | Rows    | MD5                                |
| ----------------------- | ------- | ---------------------------------- |
| `sigma-contracts.csv`   | 193,161 | `0745f38d50fdcfddb0faba6b4bc72080` |
| `sigma-authorities.csv` |   4,441 | `ec0906ae88ac616d0fb9ef2bd49d44a5` |
| `sigma-companies.csv`   |  17,455 | `249108563d39ed2f00745b7cd7d744bc` |

Snapshot details:

- Period: 2020–2026 (2026 partial, up to 16 June 2026)
- Source: АОП / ЦАИС ЕОП open data (via `storage.eop.bg`), through SIGMA
- License: CC-BY 4.0
- Currency: contract values standardised in EUR
- Access: direct CSV export from SIGMA; no public real-time API is provided as of June 16, 2026

## Sector labels (CPV reference)

The `sector_code` feature is the **CPV 2008 division** — the first two digits
of a contract's Common Procurement Vocabulary code (Regulation (EC) No 213/2008).

Human-readable names for these codes are derived in
[`notebooks/00_cpv_sector_reference.ipynb`](notebooks/00_cpv_sector_reference.ipynb),
which reads the official EU code list and writes
[`data/reference/cpv_sectors.csv`](data/reference/cpv_sectors.csv). The later
notebooks load that CSV. Every label is taken verbatim from the official file —
none are hand-written or guessed.

**Source:** Common Procurement Vocabulary (CPV) 2008, official multilingual code
list (`data/reference/cpv_2008_ver_2013.xlsx`), published by the European
Commission via TED/SIMAP (https://ted.europa.eu/en/simap/cpv). Legal basis:
Regulation (EC) No 2195/2002, as amended by Regulation (EC) No 213/2008. EU
material is reused under the Commission's reuse policy (Decision 2011/833/EU)
with attribution.

## Structure

- `report.ipynb` - project write-up: problem, framing, significance, results (root)
- `notebooks/` - the step-by-step analysis (Jupyter)
  - `00-cpv-sector-reference.ipynb` - derive CPV division labels from the official
    EU file → `data/reference/cpv_sectors.csv` (run this first)
  - `01-initial-exploration.ipynb` - target balance, the procedure trap, value skew
  - `02-cleaning-and-joins.ipynb` - cleaning, authority join → the modelling table
  - `03-exploratory-analysis.ipynb` - single-bid rate by funding, authority, region,
    value, time; the procedure circularity consolidated
  - `04-logistic-regression.ipynb` - baselines, logistic regression, odds-ratio
    coefficients, with/without direct awards
  - (later) trees / ensembles, evaluation
- `data/raw/` - the downloaded contract CSVs (not tracked; see "Obtaining the data")
- `data/reference/` - `cpv_2008_ver_2013.xlsx`, the official EU CPV 2008 code list
  (tracked); `cpv_sectors.csv` is generated by notebook `00` (not tracked)
- `data/processed/` - the cleaned modelling table, built by `02`

## License / attribution

- **Code & notebooks:** © 2026 Denis Milanov, released under the MIT License
  (see `LICENSE`).
- **Data:** Bulgarian public procurement data, CC-BY 4.0 — АОП / ЦАИС ЕОП via
  SIGMA (`https://sigma.midt.bg`). Used and redistributed under CC-BY 4.0 with
  attribution (an archived snapshot is provided via GitHub Releases for
  reproducibility).

## References - single-bid as a competition-risk indicator

- **European Commission, Single Market Scoreboard - "Access to public
  procurement."** The "Single bidder" indicator measures the proportion of
  awarded contracts that received one bid; framework agreements are excluded, and
  direct awards are excluded because their rules make no provision for
  competition. Rated green at ≤10%, red at >20%.
  https://single-market-scoreboard.ec.europa.eu/business-framework-conditions/public-procurement_en

- **OECD, Fighting Bid Rigging in Public Procurement.** Red flags are signals to
  investigate, not proof; a pattern across many contracts says more than any
  single case.
  https://www.oecd.org/en/topics/sub-issues/competition-enforcement/fighting-bid-rigging-in-public-procurement.html