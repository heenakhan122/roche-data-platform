# Roche PCR Data Platform

**Roche CSI Assay Team — Summer/Fall 2026**

I built an internal data platform to centralize PCR and HIT-PCR assay experimental data across all of Roche's diagnostic assays. The team was tracking everything in separate Excel files per assay — no cross-assay querying, no lineage, no single source of truth. This replaced all of that.

---

## The Problem

The CSI Assay Team generates a lot of experimental data across different assay types — PCR runs, HIT-PCR pool designs, in silico specificity predictions, oligo designs, PLR/EVAB reports. Each lived in its own Excel file. Cross-assay analysis meant manually stitching spreadsheets together, and there was no way to know if the data you were looking at was the latest version.

---

## What I Built

### PostgreSQL Schema
Designed a normalized schema across 25+ tables covering the full assay data lifecycle — from oligo and pool design through instrument run results.

A few decisions I'm proud of:
- **Class table inheritance** (`result → pcr_result → hit_result`) to handle shared and type-specific fields cleanly without duplication
- **Design-time vs run-time separation** — distinct tables for designed pools/templates vs actual run instances, which was a non-obvious distinction that matters a lot for this domain
- **ETL audit logging** via an `etl_run_log` table — every ingestion run is tracked with timestamps, row counts, and source file metadata
- Partial indexes on high-cardinality filter columns (e.g. QC flags) for query performance

### ETL Pipelines (Python)
Five ingestion scripts, one per report type the team produces:

| Script | Source | What it ingests |
|--------|--------|-----------------|
| `insilico_etl.py` | hgDNA + interaction reports | Human genomic DNA specificity and interaction predictions |
| `hit_report_etl.py` | 8-sheet PLR/HiT Excel reports | HIT-PCR experimental results |
| `pcr_run_etl.py` | PCR instrument CSV exports | Raw PCR run results and amplification data |
| `plr_evab_etl.py` | PLR EVAB reports | Long-amplicon PLR results |
| `pool_etl.py` | nonRT/RT pooling files | HiT pool designs — 6 pools, 1,995+ templates per pool |

All scripts are idempotent and log every run to `etl_run_log`.

### REST API (Django + Django Ninja)
Built a REST API to expose the database to downstream consumers and the frontend:
- `GET /api/projects` — all assay projects
- `GET /api/projects/{id}/runs` — runs under a project
- `GET /api/runs/{id}/results` — results for a run
- `GET /api/runs/{id}/curves` — amplification curve data
- `GET /api/etl/log` — ingestion audit log

### React Frontend
Data explorer (React + Vite) for browsing projects, runs, results, amplification curves, and ETL logs.

### AI Agent
Built a natural language query agent over the assay database — scientists can ask questions like *"which assays had >5% QC failures in the last 3 months?"* and get accurate, SQL-backed answers. Includes:
- NL-to-SQL generation grounded in the PCR schema
- An eval framework with a ground-truth Q&A set and automated scoring
- Chat UI integrated into the React frontend

---

## Stack

PostgreSQL · Python · Django · Django Ninja · React · Vite · GitHub Actions

---

## Note

This is an internal proprietary system — code and data aren't public. Happy to walk through architecture and design decisions.
