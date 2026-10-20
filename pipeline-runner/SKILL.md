---
name: pipeline-runner
description: Use when user says "run the pipeline", "research TICKER", "run pipeline for X", wants to kick off research processing, inspect a pipeline job, verify traceability, or read back a pipeline trace.
---

# Pipeline Runner

> **Notes are DB-only (kata x2cx, CIO 2026-10-14).** The note is the active
> structured document (`research.note_documents` + `research.note_sections`),
> read with `SELECT research.get_note_structured('<TICKER>', NULL)` and written
> only by `research.save_note_structured` (pipeline, or
> `research/scripts/apply_note_revision.py` for a curated revision).
> `research/tickers/*.md` are stale pipeline scratch files and
> `research.companies.research_note_body` is a stale pre-cutover snapshot:
> do not read either as the current note, do not edit or commit `.md` notes,
> and never write `research_note_body`. Any step below that reads, edits,
> creates or commits `research/tickers/*.md` is superseded until this skill is
> rewritten (kata x2cx; this skill's rewrite is part of kata sg4f).

Runs or inspects the research pipeline for one or more tickers.

Pipeline output is not complete unless source artifacts are durably persisted to R2 under the canonical company-id dossier prefix:

```text
research/company_id=.../ticker=.../date=.../run=<pipeline_job_id>/
```

Required trace events must include non-empty `source_urls` and `source_artifact_keys`, including the dossier `manifest.json`. Missing source artifact persistence is a blocking defect, not a warning.

## Single Ticker
For an ad-hoc local draft with no governed readback requirement:

```bash
cd "$(git rev-parse --show-toplevel)/research/pipeline" && unset PYTHONPATH && \
  bws run -- uv run work --ticker BSRR
```

For any request that must satisfy trace/readback validation, enqueue an actual queue-backed job first through internal-api `POST /api/v1/rawls/research/pipeline-job` or Port, then validate the returned `pipeline_job_id` through the trace endpoint below. (The `/rawls/` in these paths is a legacy route name; internal-api serves them, not the decommissioned host.) The local `uv run orchestrate` command scans and mutates the markdown work queue directly; it does not create a `research.pipeline_jobs` row or return a traceable `pipeline_job_id`. Do not require job-trace readback for direct `uv run work --ticker ...` or local `uv run orchestrate ...` runs unless they were attached to a queue job.

## Local Batch Refresh (no job-trace readback)
Do not use the unfiltered default `uv run orchestrate --max-cost ...` form for routine refreshes: it can select review/exclusion-state notes. The orchestrator also drains the investigation queue before the work queue, so first run a dry run and do not start held/pipeline refresh passes while investigation items are pending.

Once the investigation queue is empty, use explicit held/pipeline passes:

```bash
cd "$(git rev-parse --show-toplevel)/research/pipeline" && unset PYTHONPATH && \
  bws run -- uv run orchestrate --status held --max-cost 20.0
cd "$(git rev-parse --show-toplevel)/research/pipeline" && unset PYTHONPATH && \
  bws run -- uv run orchestrate --status pipeline --max-cost 20.0
```

If investigation items are pending and a specific ticker must be refreshed anyway, use a targeted internal-api/Port enqueue instead of a broad local orchestrator refresh. Do not include `needs_review` or `excluded_candidate` without CIO/exclusion-review intent.

For agent orchestration, use the internal-api contracts instead of deprecated `ecic` paths:

- `POST /api/v1/rawls/research/pipeline-job` to enqueue.
- `GET /api/v1/rawls/research/pipeline-job/:id` for canonical job status only.
- `GET /api/v1/rawls/research/pipeline-job/:id/trace` for trace events and traceability validation; fail fast when `traceability.ok` is false.
- Port remains the human operator surface.

## Resume or Restart a Local Batch
`orchestrate` resumes prior progress by default. To ignore prior progress and start over, keep the same explicit status filter:

```bash
cd "$(git rev-parse --show-toplevel)/research/pipeline" && unset PYTHONPATH && \
  bws run -- uv run orchestrate --status held --max-cost 20.0 --no-resume
```

## After Pipeline Completes
1. Review the batch summary printed to stdout.
2. For queue-backed jobs, validate canonical job state and traceability through `/api/v1/rawls/research/pipeline-job/:id/trace` or Port.
3. For queue-backed jobs, confirm source-bearing trace events reference the R2 dossier manifest and source artifacts.
4. Commit remaining notes from the repo root: `git -C "$(git rev-parse --show-toplevel)" add research/tickers/ && git -C "$(git rev-parse --show-toplevel)" commit -m "research: pipeline batch $(date +%Y-%m-%d)"`.
5. Push your branch through the Forgejo wrapper and open a PR (`CLAUDE.md` worktree section); never bare `git push`.

## Key Facts

- Auto-syncs to DB + git commits every 25 notes during batch runs
- Source evidence dossiers live under `research/company_id=.../ticker=.../date=.../run=<pipeline_job_id>/`
- Cost tracking is live via DeepInfra pricing API
- Budget tiers: full (tick>=10), comprehensive (5-9), moderate (1-4), maintenance (<=0)
- Models: DeepSeek-V3.2 (fast), Kimi-K2 (smart), GLM-5 (final)
- ~$0.01 per ticker, ~$13-16 for full universe
- `unset PYTHONPATH` is required (prevents venv conflicts)
- Do not use deprecated `ecic` as a new research-pipeline agent surface
