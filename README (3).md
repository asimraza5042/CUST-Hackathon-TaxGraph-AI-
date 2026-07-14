# TaxGraph AI — Pakistan FBR Intelligence Platform

A proof-of-concept pipeline and interactive dashboard that demonstrates how records
from disconnected government databases (NADRA, Excise, WAPDA, FBR) can be resolved
into unified taxpayer entities, linked in a knowledge graph, and scored for tax
compliance deviation — with a full explainable audit trail behind every score.

> ⚠️ **Synthetic data only.** All names, CNICs, addresses, and records in this repo
> are fabricated for demonstration purposes and do not represent real individuals.

---

## What it does

1. **Entity Resolution** — fuzzy-matches people across four independent sources
   (name similarity, CNIC-prefix anchoring, address similarity) to cluster records
   that refer to the same person, even when names/addresses are recorded
   inconsistently across agencies.
2. **Knowledge Graph Construction** — builds a directed graph (Person → Vehicle,
   Person → Meter, Person → Tax Filing) using NetworkX, capturing how each
   individual is cross-linked across databases.
3. **Tax Compliance Deviation Scoring** — estimates a "lifestyle income" proxy from
   owned vehicle value and electricity consumption, compares it against declared
   income, and produces a 0–100 deviation score with plain-language audit reasons
   for every flag (explainable, not a black box).
4. **Interactive Dashboard** — a self-contained HTML app for exploring entities,
   evidence chains, the knowledge graph, and score methodology.

---

## Project structure

```
├── pipeline.py                    # Main pipeline: entity resolution → graph → scoring
├── TaxGraph_Pipeline.ipynb        # Same pipeline as a runnable Jupyter notebook
├── generate_entities_js.py        # Converts pipeline output into dashboard.html's data source
├── dashboard.html                 # Self-contained interactive dashboard (open in a browser)
├── entities_data.js               # Generated data consumed by dashboard.html
│
├── nadra.csv                      # Source: national identity records
├── excise.csv                     # Source: vehicle registration records
├── wapda.csv                      # Source: electricity utility billing records
├── fbr.csv                        # Source: tax filing records
│
├── entity_resolution_output.json  # Pipeline output: resolved, cross-linked entities
├── deviation_scores_output.json   # Pipeline output: scored entities + audit reasons
├── deviation_scores.csv           # Pipeline output: flat summary table
└── knowledge_graph.gexf           # Pipeline output: graph (open in Gephi)
```

## Data flow

```
nadra.csv  ─┐
excise.csv ─┼──▶ pipeline.py ──▶ entity_resolution_output.json ─┐
wapda.csv  ─┤        │                                          ├──▶ generate_entities_js.py ──▶ entities_data.js ──▶ dashboard.html
fbr.csv    ─┘        ├──▶ deviation_scores_output.json ─────────┘
                      ├──▶ deviation_scores.csv
                      └──▶ knowledge_graph.gexf
```

---

## Setup

```bash
pip install pandas networkx
```

Requires Python 3.9+.

## Running the pipeline

```bash
python pipeline.py
```

This reads the four source CSVs and regenerates:
- `entity_resolution_output.json`
- `deviation_scores_output.json`
- `deviation_scores.csv`
- `knowledge_graph.gexf`

Alternatively, open and run `TaxGraph_Pipeline.ipynb` top to bottom for the same
result with inline output at each step.

## Regenerating the dashboard data

After running the pipeline, rebuild the dashboard's data file:

```bash
python generate_entities_js.py
```

This reads `entity_resolution_output.json` and `deviation_scores_output.json` and
writes `entities_data.js`, which `dashboard.html` loads directly.

## Viewing the dashboard

`dashboard.html` is fully self-contained (no server or build step needed) as long
as `entities_data.js` sits next to it — just open `dashboard.html` in a browser.

The dashboard includes:
- **Overview** — summary stats, risk distribution, and score/source charts
- **Entities** — searchable list of resolved taxpayers with risk bands (critical / high / medium)
- **Entity deep-dive** — declared vs. lifestyle income, linked assets, source-by-source evidence table
- **Audit trail** — plain-language, source-cited reasons behind each deviation score
- **Knowledge graph** — visual explorer of Person/Vehicle/Meter/Filing linkages
- **Methodology** — how entity resolution and scoring are calculated

---

## Methodology notes

- **Entity resolution** combines CNIC-prefix matching (50%), name similarity via
  `difflib.SequenceMatcher` (30%), and address similarity (20%) for vehicle
  records, using a 0.55 confidence threshold; WAPDA records use a 60/40 name/address
  weighting with a 0.5 threshold.
- **Lifestyle income proxy** = 15%/year of total registered vehicle value +
  (avg. monthly electricity bill × 12 × 4).
- **Deviation score** = `max(0, (lifestyle_income − declared_income) / lifestyle_income) × 100`,
  capped at 100.
- Flags such as "non-filer owns high-value vehicles" or "electricity bill
  inconsistent with declared income" are generated directly from the same
  thresholds used to compute the score, so every number on the dashboard traces
  back to a specific rule and source record.
- The scoring logic here is a **heuristic stand-in** for a trained anomaly-detection
  model (e.g., a GNN), intended to demonstrate the explainability layer rather than
  serve as a production risk model.

## Limitations

- Dataset is small (10 synthetic individuals) and intended for demonstration, not
  statistical validity.
- Matching thresholds are illustrative and would need tuning/validation against
  real, larger-scale data before any operational use.
- This is a proof of concept, not an audited or production-ready compliance tool.
