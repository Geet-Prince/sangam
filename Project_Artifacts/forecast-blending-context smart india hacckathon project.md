# Hybrid AI–NWP Forecast Blending — Project Context

## 1. Problem Statement
- **ID:** 26081
- **Org:** Ministry of Earth Sciences (MoES) / NCMRWF
- **Category:** Software | **Theme:** Disaster Management
- **Goal:** dynamically blend multiple forecast sources (NWP, ensemble, AI/ML
  models) using adaptive weights based on historical skill, lead time,
  region, season, and weather regime — for rainfall, temperature, wind, and
  extreme weather indicators.

## 2. Scope
This is **not** a from-scratch weather prediction model (that needs
supercomputer-scale NWP or GraphCast-class compute/data, out of reach for
this project). It's a decision/blending layer on top of forecasts that
already exist.

Three deliverables:
1. A blending model that learns which source to trust, when, where
2. An automated ingestion + skill-scoring pipeline
3. A dashboard showing the blended forecast, model weight maps, and
   extreme-weather flags

## 3. Architecture

```
External sources (IMD, GFS, ECMWF, open AI models)
        │
        ▼
Ingestion & preprocessing (scheduled jobs, normalize GRIB/NetCDF)
        │
        ▼
Skill scoring & feature store (accuracy by region / lead time / season / regime)
        │
        ▼
Adaptive blending model (learns per-source weights per condition)
        │
        ▼
API layer (serves blended forecast + weight maps)
        │
        ▼
Dashboard (visualizes forecast, weight maps, extreme alerts)
```

## 4. Tech Stack & Rationale

| Layer | Choice | Why |
|---|---|---|
| Blending model | LightGBM / XGBoost | Fast to train, no GPU, explainable per-condition weights — a neural net is unjustified overhead here |
| Backend / API | FastAPI | Same Python as Flask experience, but async + auto OpenAPI docs, lighter for ML serving |
| Database | Supabase (Postgres) | Already in use; stores raw forecasts, skill scores, blended outputs |
| Frontend | Astro or Next.js, React islands for interactivity | SSR/SSG makes public pages crawlable — plain Vite/CSR ships an empty `<div>` that hurts SEO |
| Charts / maps | Plotly.js or Recharts + Leaflet | Lightweight, no heavy GIS stack |
| Scheduling | cron / GitHub Actions | Airflow-class orchestration is unjustified weight at this scale |
| Hosting | Vercel (frontend) + Render/Railway (API) | Free tiers, minimal ops |

**Locked constraints:** lightweight, SEO-friendly where public-facing, free/open tooling only.

## 5. Agent Workflow Rule (for OpenCode via `AGENTS.md`)

**Why:** an agent working autonomously across ML + API + frontend can
silently violate the two hard constraints above — e.g. pull in a heavy
dependency for a small feature, or regress a page back to client-only
rendering — without it showing up until a much later diff review.

**Before acting:**
- State a one-line plan and which architecture layer it touches
- Flag any new dependency before adding it — check it against "lightweight"

**After acting:**
- Diff `package.json` / `requirements.txt` for anything new and justify it
- Confirm public pages are still SSR/SSG (no CSR regression)
- Run build/lint
- Summarize what changed vs. what's unverified

**Why this is better than a general "be careful" instruction:** each
checkpoint is checkable, not judgment-based — a new dependency either
appeared in the diff or it didn't; a page either still renders server-side
or it doesn't. Drift gets caught at the action that caused it, not three
sessions later as unexplained scope creep.
