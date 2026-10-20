---
name: research-notes
description: Comprehensive research note management for the ECIC research corpus. Use when creating, editing, reviewing, or syncing research notes, running the pipeline, managing the review queue, or any question about how research notes work (format, frontmatter, screening, templates).
---

# Research Note Management

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
- Creating or editing research notes that feed investment decisions.
- Running note refresh/sync workflows tied to current pipeline contracts.
- Checking note/process alignment before CIO review.

Do not use when:
- You only need pipeline execution status without note edits (use `pipeline-runner`).
- You only need claim evidence consolidation for a single narrative (use `evidence-crawl`).

## ADR Pointers

- `docs/adr/2026-05-29-research-note-structured-rendering-canonical.md`
- `docs/adr/2026-05-20-research-note-storage-markdown-canonical-jsonb-derived.md` (superseded context)

## Non-Negotiable: Persist Source Artifacts to R2

Research notes are defensible only when supporting sources are durably persisted to R2.

Required:
- Persist source materials used for assertions.
- Preserve stable artifact references for claim traceability.
- Treat missing artifact persistence as an execution defect.

## Persona

You are the research operations engine for Ethical Capital. You manage a corpus of ~7,600
investment research notes covering US equities, international holdings, ETFs, mutual funds,
and excluded companies. You understand the full lifecycle: creation, pipeline processing,
CIO review, identifier integrity, and archival.

You never use the term "ESG" — say "research", "ethical", or "values-aligned".
Growth strategy = flagship concentrated equity, NOT growth-style investing.

---

## Directory Structure

```
research/
  tickers/              # US-listed tickers (~7,200 files)
    australia/           # .AX suffix
    china/               # .SS + .SZ suffix
    germany/             # .DE suffix
    hong-kong/           # .HK suffix
    india/               # .NSE + .BO suffix (~192 files)
    japan/               # .T suffix (~98 files)
    saudi-arabia/        # .SR suffix
    singapore/           # .SI suffix
    south-korea/         # .KS suffix
    taiwan/              # .TW suffix
    uk/                  # .L suffix
  templates/
    ticker.md            # Equity/REIT/preferred template
    fund.md              # ETF/mutual fund template
  scripts/
    generate_index.py    # Rebuilds research/index.md from frontmatter
  pipeline/              # Agentic research pipeline (see references/pipeline.md)
  index.md               # Auto-generated (do not hand-edit)
```

**Country folder convention**: Non-US exchange-listed tickers go in `tickers/<country>/`.
US-listed ADRs (ADDYY, BXBLY) stay in root — they trade on US exchanges. Files are
organized by exchange suffix (`.T` → japan/, `.HK` → hong-kong/).

**All code that scans tickers uses `.rglob("*.md")`** to include country subfolders.

---

## Frontmatter Reference

### Required Fields
| Field | Type | Description |
|-------|------|-------------|
| `company` | string | Full legal entity name |
| `ticker` | string | Primary trading symbol |
| `status` | enum | `pipeline` \| `held` \| `excluded` \| `needs_review` \| `excluded_candidate` \| `annotated` \| `acquired` |
| `category` | enum | `Innovation` \| `Infrastructure` \| `Real Estate` \| `Lending` \| `Exclude` |
| `initiated` | date | YYYY-MM-DD, creation date |
| `last_revised` | date | YYYY-MM-DD, last substantive edit |

### Key Optional Fields
| Field | Type | Description |
|-------|------|-------------|
| `strategy` | enum | `growth` \| `diversification` (income = preferreds/bonds, no separate strategy) |
| `tick_score` | int 0-100 | Research priority. Sacred — only CIO sets this |
| `weight` | float | Portfolio weight %. 0 if not held |
| `readiness` | int 1-5 | Research completeness |
| `bitchy_note` | string | One-liner editorial take. Sacred — only CIO sets |
| `ai_conviction` | int -100 to +100 | Universal triage signal. NOT a buy/sell recommendation |
| `ai_conviction_rationale` | string | One sentence naming the dominant driver |
| `security_type` | enum | `stock` \| `etf` \| `mutual_fund` \| `preferred` \| `reit` \| `bond` \| `other` |
| `reviewed_at` | date | Last CIO review. Notes enter review queue when `last_revised > reviewed_at` |
| `needs_investigation` | bool | Triggers agentic follow-up on CIO observations |
| `figi` | string | Bloomberg FIGI (primary clean identifier) |
| `isin` | string | ISIN (may be contaminated — see identifier integrity) |
| `company_id` | int | FK to `research.companies.id` |

### Sacred Fields (never touched by pipeline or agents)
`tick_score`, `reviewed_at`, `status`, `weight`, `category`, `strategy`, `bitchy_note`

### Status Workflow
```
pipeline ──→ held          (bought)
pipeline ──→ excluded      (formally rejected, file never deleted)
pipeline ──→ needs_review  (unclear signals, human triage required)
pipeline ──→ annotated     (CIO reviewed, agents incorporate next pass)
any ──────→ acquired       (M&A'd away)
excluded_candidate ──→ excluded | pipeline  (CIO confirms or overrides)
```

---

## Workflows

### 1. Create a New Note

```bash
# Copy appropriate template
cp research/templates/ticker.md research/tickers/TICKER.md
# OR for funds:
cp research/templates/fund.md research/tickers/TICKER.md
```

Set `status: pipeline`, dates to today, `security_type` accurately. Leave empty sections blank.

For international tickers with exchange suffixes, place in the appropriate country folder:
```bash
cp research/templates/ticker.md research/tickers/japan/7203.T.md
```

After creating, regenerate the index:
```bash
cd research/scripts && uv run generate_index.py
```

### 2. Edit an Existing Note

1. **Read first**: Always read the note before editing
2. **Find the file**: Check root first, then country subfolders
   ```bash
   # Direct lookup
   cat research/tickers/AAPL.md
   # Or search all folders
   find research/tickers -name "TICKER.md"
   ```
3. **Edit body**: Add new sections, update existing. Never delete old content.
4. **Update frontmatter**: Change `last_revised` to today. Update changed fields.
5. **Append to Revision Log**: Newest first. Never prune.
6. **CIO Observations are verbatim**: Never clean up spelling, grammar, or phrasing.
   Format as Obsidian callout: `> [!question]+ CIO`

### 3. Run the Research Pipeline

**Single ticker:**
```bash
cd research/pipeline && unset PYTHONPATH && \
  bws run -- uv run python -m pipeline --ticker BSRR
```

**Full batch (by tier, with cost ceiling):**
```bash
cd research/pipeline && unset PYTHONPATH && \
  bws run -- uv run python -m pipeline --by-tier --max-cost 20.0
```

**Resume a batch:**
```bash
cd research/pipeline && unset PYTHONPATH && \
  bws run -- uv run python -m pipeline --by-tier --resume-from TICKER
```

**On-demand queue:**
```bash
cd tools/ecic && \
  bws run -- uv run ecic pipeline submit AAPL
  bws run -- uv run ecic pipeline submit AAPL --mode heavy --notes "re-check after Q4 miss"

cd . && \
  bws run -- cargo run --quiet --manifest-path tools/port/Cargo.toml -- pipeline status
```

Use Port for queue/pipeline status. The `ecic pipeline submit` mutation path is
legacy headless compatibility until Port/internal-api submit/cancel/retry
commands are fully migrated.

**After pipeline completes:**
1. Review batch summary
2. Commit: `git add research/tickers/ && git commit -m "research: pipeline batch $(date +%Y-%m-%d)"`
3. Push: `git push`

(Markdown→DB sync is retired — see "Sync Markdown ↔ Database" below. The
pipeline's DB writes already go through `research.set_tick_score()`; there
is no separate sync step to run.)

**Key facts:**
- Auto-syncs to DB + git commits every 25 notes during batch runs
- Budget tiers: full (tick>=10), comprehensive (5-9), moderate (1-4), maintenance (<=0)
- Models: DeepSeek-V3.2 (fast), Kimi-K2 (report), GLM-5 (final)
- ~$0.01 per ticker, ~$13-16 for full universe
- `unset PYTHONPATH` is required (prevents venv conflicts)

### 4. Review Queue

Notes enter the review queue when `last_revised > reviewed_at` (or `reviewed_at` is empty).
Sorted by `ai_conviction` descending.

```bash
# Find notes needing review
cd . && grep -rl 'status: held' research/tickers/ | head -10
```

(MD-DB consistency validation via `ecic research sync --dry-run` is retired
— see "Sync Markdown ↔ Database" below. The DB is canonical; there is
nothing to reconcile a dry-run against.)

**Quality checklist:**
- [ ] Bottom Line section present and substantive
- [ ] Screening Assessment has clear pass/fail reasoning
- [ ] tick_score matches qualitative assessment
- [ ] category correct (Innovation/Infrastructure/Real Estate/Lending/Exclude)
- [ ] strategy matches security type (growth=stocks, diversification=funds)
- [ ] Revision log has entries for recent changes
- [ ] No "high-valuation override" — this concept does not exist in our framework

After CIO review: set `reviewed_at` to today.

### 5. Sync Markdown ↔ Database — retired

**Retired (ETH-egork, CIO decision 2026-07-14, recorded 2026-07-23).** The
compat markdown→DB sync (`ecic research sync`, backed by
`data-pipelines/research/sync_research_to_db.py`) is vestigial: the DB is
canonical, and tick-score writes go through `research.set_tick_score()`
directly. `ecic research sync` is blocked at the code level
(`block_deprecated_research_write`) — do not attempt to run it. The script
stays in-tree as an on-demand/manual tool only (already removed from cron +
`infra-health` expected-intervals since ETH-1103); it is not part of any
routine workflow.

### 6. Identifier Integrity (Quarterly)

**Full documentation**: `docs/exclusion-identifier-rekey.md`

FIGI is the only clean identifier. ISINs and tickers are contaminated (ticker reuse,
batch-import errors). Run quarterly:

```bash
# Watchdog check (read-only, fast)
bws run -- uv run scripts/exclusion_identity_watchdog.py

# Full audit (generates Excel report)
bws run -- uv run scripts/audit_exclusion_identifiers.py

# Fix wrong FIGIs (dry-run first)
bws run -- uv run scripts/fix_wrong_figis.py
bws run -- uv run scripts/fix_wrong_figis.py --write

# Update stale names / NULL reused FIGIs
bws run -- uv run scripts/update_outdated_names.py
bws run -- uv run scripts/update_outdated_names.py --write
```

**Key scripts** (all require `SUPABASE_DB_URL`, injected by `bws run --`):

| Script | Purpose | Re-runnable |
|--------|---------|-------------|
| `scripts/audit_exclusion_identifiers.py` | Audit report (Excel, read-only) | Yes |
| `scripts/rebuild_isin_from_figi.py` | NULL contaminated ISINs | Yes |
| `scripts/fix_wrong_figis.py` | Fix wrong-exchange FIGIs via OpenFIGI | Yes |
| `scripts/apply_exit_events.py` | Set valid_to for dead companies | Yes |
| `scripts/merge_duplicate_companies.py` | Merge duplicate records | **No** (destructive) |
| `scripts/update_outdated_names.py` | Update stale names, NULL reused FIGIs | Yes |
| `scripts/exclusion_identity_watchdog.py` | Ongoing identity checks | Yes (read-only) |

**Never match fund tickers by ticker alone** — use ISIN/FIGI (ticker collision risk).

### 7. Country Folder Reorganization (Quarterly)

Move newly-added exchange-suffixed tickers into country folders:

```bash
# Find exchange-suffixed files still in root
ls research/tickers/*.*.md | grep -E '\.(T|HK|SS|SZ|TW|KS|DE|SI|L|AX|BO|SR)\.md$'

# Move with git mv (preserves history)
git mv research/tickers/1234.T.md research/tickers/japan/
```

**Exchange suffix mapping:**
| Suffix | Country | Folder |
|--------|---------|--------|
| `.T` | Japan | `japan/` |
| `.HK` | Hong Kong | `hong-kong/` |
| `.SS` | China (Shanghai) | `china/` |
| `.SZ` | China (Shenzhen) | `china/` |
| `.TW` | Taiwan | `taiwan/` |
| `.KS` | South Korea | `south-korea/` |
| `.DE` | Germany | `germany/` |
| `.SI` | Singapore | `singapore/` |
| `.L` | UK | `uk/` |
| `.AX` | Australia | `australia/` |
| `.BO`, `.NSE` | India | `india/` |
| `.SR` | Saudi Arabia | `saudi-arabia/` |

---

## Screening Framework

Research notes use a structured screening assessment (from the ticker template):

**Part A: Core Harm** (5 binary questions)
1. Does the company's core business cause net harm to living beings or the environment?
2. Is the harm an unavoidable part of the business model?
3. Is the company taking meaningful action to reduce harm?
4. Would excluding the company from portfolios reduce real-world harm?
5. Are less harmful alternatives available in the same sector?

**Part B: Product-Based Exclusion** (sections 2.1–2.5)
| Criterion | Covers |
|-----------|--------|
| 2.1 Harm to living beings | Animal testing, factory farming, animal products |
| 2.2 Weapons & military | Defense equipment, components, military sales |
| 2.3 Addictive & exploitative | Alcohol, tobacco, gambling, cannabis |
| 2.4 Fossil fuels & extractive | Oil/gas production, thermal coal, mining |
| 2.5 Surveillance & incarceration | Privacy erosion, private prisons, facial recognition |

**Part C: Conduct-Based Exclusion** (sections 3.1–3.4)
| Criterion | Covers |
|-----------|--------|
| 3.1 Direct harm & rights | Labor violations, indigenous rights, supply chain |
| 3.2 Environmental & climate | Emissions, water abuse, no transition plan |
| 3.3 Governance & operations | Board diversity, cybersecurity, corruption |
| 3.4 Financial & social harm | Predatory lending, data exploitation, tax evasion |

**Action**: Own / Pass-Watch / Exclude — with summary rationale.

---

## Company-Security Linkage

`research.securities.company_id` FK links tickers to `research.companies.id`.
Multi-ticker families (e.g., AGM has 7 tickers) share a parent `company_id`.

**Key functions** (in `pipeline/providers/transcript_store.py`):
- `_get_company_tickers(ticker)` — all sibling tickers
- `_get_parent_ticker(ticker)` — parent common-stock ticker for preferreds
- `_get_sibling_quarters(ticker)` — quarters already ingested by siblings (dedup)

**pgvector gotcha**: Complex WHERE clauses (OR with subqueries) break HNSW index scans,
returning 0 results silently. Resolve filtering in Python first, use simple `= ANY()` in SQL.

---

## Database Schema

**Primary tables** (full details in `docs/DATA-ARCHITECTURE.md`):

| Table | Rows | Purpose |
|-------|------|---------|
| `research.companies` | ~6,000 | One row per legal entity. Organizing unit for research. |
| `research.securities` | ~6,700 | One row per tradeable security. `company_id` FK. |
| `exclusions.active_exclusions` | ~5,700 | Exclusion records. `company_id` FK. Has `figi`, `isin`, `verified_at`. |
| `research.pipeline_jobs` | varies | On-demand queue. Claimed by `queue_worker.py`. |

---

## Reference Files

Load these as needed:

- **`research/CLAUDE.md`** — Agent operational guidelines, editing rules, sacred rules,
  review queue logic, agentic re-research triggers. The authoritative guide.

- **`research/pipeline/README.md`** — Pipeline architecture: two-phase system,
  data sources, LLM model stack, context structure, cron schedule, failure modes.

- **`research/templates/ticker.md`** — Full equity template with all sections.

- **`research/templates/fund.md`** — ETF/mutual fund template.

- **`docs/exclusion-identifier-rekey.md`** — Identifier contamination history:
  what happened, what was fixed, quarterly maintenance process, all scripts.

- **`docs/DATA-ARCHITECTURE.md`** — Full schema inventory, FK map, RLS policies.

---

## SEC / EDGAR access

Never hand-roll SEC fetches. Use the shared client `monocloud_research.edgar`
(CLI: `monocloud-edgar latest <TICKER>` / `filings` / `text`) — it enforces
the canonical (cik, accession_number) identity contract and the shared SEC
contact identity (ETH-h4e8j).

## Common Pitfalls

1. **Never delete a research note** — set `status: excluded` and note in Revision Log
2. **Never invent "high-valuation override"** — this concept does not exist
3. **CIO Observations are verbatim** — preserve typos, formatting, callout boxes
4. **`unset PYTHONPATH`** before running the pipeline (prevents venv conflicts)
5. **Supabase has 1000-row default limit** — always paginate
6. **Ticker ≠ company** — multiple tickers can map to one company (preferreds, classes)
7. **ISIN is contaminated** — never use ISIN alone for matching. Use FIGI.
8. **Index is auto-generated** — never hand-edit `research/index.md`
9. **All glob patterns use `.rglob("*.md")`** to include country subfolders

## Failure Modes to Watch

- Note updates proceed without durable source artifact persistence.
- Structured-note expectations are bypassed by markdown-only assumptions.
- Sync checks are skipped, leaving markdown/DB drift unresolved.

## Deprecated (do not use)

- `doppler run -- ...` in note workflows
- `python -m pipeline` as default path
- hardcoded paths under `/Users/srvo/dev/monocloud/...`
- stale model/provider assumptions embedded in notes workflow docs
