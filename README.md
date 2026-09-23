# Roche Data Platform

**Roche — Summer/Fall 2026**

Internal data platform I built to centralize experimental data across multiple teams. Replaced fragmented spreadsheet-based tracking with a normalized database, automated ingestion pipelines, a REST API, and an AI query layer.

---

## Stack

PostgreSQL · Python · Django · Django Ninja · React · Vite · GitHub Actions

---

## What I Built

### Database Schema
Designed a normalized PostgreSQL schema across 25+ tables. Key decisions:
- **Class table inheritance** to handle shared and type-specific fields across related entities without duplication
- **Design-time vs run-time separation** — distinct tables for planned records vs actual instances
- **Audit logging** via a dedicated log table — every ingestion run is tracked with timestamps, row counts, and source file metadata
- Partial indexes on high-cardinality filter columns for query performance

### ETL Pipelines
Five Python ingestion scripts covering all data source formats the team produces. All scripts are idempotent and write to the audit log on every run.

### REST API
Django + Django Ninja API exposing the database to the frontend and downstream consumers. Endpoints for browsing projects, runs, results, and ingestion history.

### React Frontend
Data explorer (React + Vite) for browsing records, results, and ETL audit logs.

### AI Agent
Natural language query agent over the database — users can ask plain English questions and get accurate, SQL-backed answers. Built NL-to-SQL generation grounded in the schema, an eval framework with a ground-truth Q&A set and automated scoring, and a chat UI in the React frontend.

---

## Note

Proprietary internal system — code and data aren't public. Happy to walk through architecture and design decisions.
