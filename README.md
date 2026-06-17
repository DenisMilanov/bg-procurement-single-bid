# Predicting Single-Bid Public Procurement Contracts

Can a contract's characteristics predict whether it will attract only a
**single bid** — a recognised competition red-flag indicator (not a measure
of corruption)? This project builds a classifier on Bulgarian public
procurement data and examines which characteristics drive the pattern.

The unit of analysis is the **contract**, not the company.

## Data

The project uses Bulgarian public procurement data from **SIGMA** — the public transparency platform for Bulgarian public procurement:

* `https://sigma.midt.bg/contracts`
* `https://sigma.midt.bg/authorities`
* `https://sigma.midt.bg/companies`

SIGMA consolidates open data from the official Bulgarian Public Procurement Register, **АОП / ЦАИС ЕОП**, made available through `storage.eop.bg`.

The analysis combines three CSV exports:

* **Contracts** — awarded public procurement contracts and lots. This is the main dataset and the unit of analysis for the model.
* **Authorities** — contracting authorities / institutions that awarded at least one procurement contract.
* **Companies** — suppliers / contractors that won at least one procurement contract.

Although authority-level and company-level information is used as additional context, the prediction task is defined at the **contract / lot level**, not at the company level. The model predicts whether a given contract attracted only a **single bid**, using contract characteristics and related metadata.

The CSV files are not tracked in this repository because of their size. To obtain the data:

1. Go to `https://sigma.midt.bg/contracts`

2. Click **“Изтегли CSV”** / **“Download CSV”**

3. Repeat the same for:

   * `https://sigma.midt.bg/authorities`
   * `https://sigma.midt.bg/companies`

4. Save the files locally, for example as:

```text
data/raw/contracts.csv
data/raw/authorities.csv
data/raw/companies.csv
```

Suggested local folder structure:

```text
data/
  raw/
    contracts.csv
    authorities.csv
    companies.csv
  processed/
notebooks/
```

Current SIGMA coverage at the time of download:

* Period: 2020–2026, with 2026 partial (up to June 16)
* Source: АОП / ЦАИС ЕОП open data via SIGMA
* License: CC-BY 4.0
* Currency: contract values standardised in EUR
* Access method: CSV export from SIGMA; no public real-time API is currently provided


## Structure

- `notebooks/` — analysis (Jupyter)
- `data/`

## License / attribution

Data: CC-BY 4.0, АОП / ЦАИС ЕОП via SIGMA.