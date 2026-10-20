---
name: research-reviewer
description: Use when reviewing research note quality, freshness, and evidence integrity before CIO decisions or downstream publishing. Triggers on "review TICKER", "check note quality", "what needs review".
---

# Research Note Reviewer

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

## When to Use

Use when:
- Assessing whether a research note is decision-ready for the CIO.
- Auditing corpus freshness or stub/low-signal coverage.
- Checking that decision-critical claims are artifact-backed.

Do not use when:
- You are creating or editing note content (`research-notes`).
- You only need pipeline execution controls (`pipeline-runner`).

## ADR Pointers

- `docs/adr/2026-05-27-canonical-artifacts-activity-annotations-schema.md`
- `docs/adr/2026-05-29-internal-services-route-through-internal-api.md`

## Non-Negotiable: R2 Artifact Auditability

Decision-critical claims must trace to a retrievable R2 artifact. A note that
reads well but cites nothing durable has not passed review.

## Review Scope

- Read notes under `research/tickers/**` — the corpus is **recursive** and
  includes country subfolders.
- Validate frontmatter and status semantics.
- Validate research quality and screening logic.
- Validate claim↔artifact traceability.

## Single-Note Review Flow

1. Read `research/tickers/**/TICKER.md`.
2. Validate required frontmatter: `company`, `ticker`, `status`, `category`,
   `initiated`, `last_revised`.
3. Check for stub/low-signal indicators: "No coverage yet", body under ~500
   characters, missing sections.
4. If `ai_conviction` is present, verify the rationale actually supports it.
5. Flag stale notes — `last_revised` more than 180 days old.
6. Verify key claims can be traced to R2-backed evidence artifacts.
7. Record pass/fail with concrete remediation.

## Finding Notes That Need Review

Run from the repo root. Globs are recursive on purpose — a root-only
`research/tickers/*.md` scan silently skips every country subfolder.

```bash
# Review queue: notes revised since the last CIO review
grep -rl 'reviewed_at:' research/tickers/ | head -5
# then compare last_revised > reviewed_at

# Held positions (priority review)
grep -rl 'status: held' research/tickers/
```

```bash
# Stale notes (>180 days since last revision)
cd research/pipeline && bws run -- uv run python -c "
from datetime import datetime, timedelta

from pipeline.providers.research_store import load_universe

cutoff = (datetime.now() - timedelta(days=180)).strftime('%Y-%m-%d')
for t in load_universe():
    if t.get('last_revised') and t['last_revised'] < cutoff:
        print(f\"{t['ticker']:8s} score={t.get('tick_score', 0):3d}  last={t['last_revised']}\")
"
```

## Consistency + Queue Checks

`ecic research sync` is retired compat tooling (ETH-egork, CIO decision
2026-07-14, recorded 2026-07-23) — the DB is canonical via the
`research.set_tick_score()` RPC, and the command is blocked at the code level
(`block_deprecated_research_write`). There is no MD-DB consistency check to
run; do not reach for one.

## Quality Checklist

- [ ] Bottom Line section present, substantive, and decision-useful.
- [ ] Screening Assessment has explicit, coherent pass/fail reasoning.
- [ ] `tick_score` matches the qualitative assessment.
- [ ] `category` is correct (Innovation / Infrastructure / Real Estate /
      Lending / Exclude).
- [ ] `strategy` matches security type — growth = common/international equity,
      income = preferreds/bonds only, diversification = funds/ETFs only.
- [ ] Frontmatter state is internally consistent.
- [ ] Revision log reflects meaningful recent changes.
- [ ] Decision-critical claims are artifact-backed in R2.

## Failure Modes to Watch

- Review signs off on quality without verifying R2 evidence linkage.
- Corpus scans miss country subfolders through non-recursive globs.
- Sync drift is ignored after a quality pass.

## Deprecated (do not use)

- Non-recursive root-only scans like `research/tickers/*.md`
- `doppler run -- ...` command examples — this repo uses `bws run --`
- Hardcoded machine paths under `/Users/...`
