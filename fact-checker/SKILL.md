---
name: fact-checker
description: Use to classify claims, verify evidence, and apply approved claim-level edits while preserving author voice and auditability. Triggers on "fact-check this", "is this true", "check this note", or auditing AI-generated content for hallucinations before publishing.
---

# Fact Checker

Five phases. Phases 1–4 run automatically; Phase 5 is the interactive loop with Sloane.

**Tools used**: Read (to read source files), WebSearch (primary web search), WebFetch (fetch specific URLs), Edit (apply approved changes), Bash (run the SearXNG/EXA helper script for parallel API searches).

---

## When to Use

Use when:
- Reviewing publishable or decision-relevant factual claims.
- Auditing drafts for hallucination or unsupported assertions.
- Preparing client-facing or compliance-sensitive text for release.

Do not use when:
- You only need broad editorial refinement without claim verification.
- You need pipeline/job execution behavior checks.

## ADR Pointers

- `docs/adr/2026-05-29-internal-services-route-through-internal-api.md` (mutation/audit boundary mindset)
- `docs/adr/2026-05-27-canonical-artifacts-activity-annotations-schema.md` (artifact governance context)

## Non-Negotiable: Evidence Persistence to R2

For decision-relevant or publishable factual claims, supporting source artifacts should be persisted to R2 (or explicitly linked to already-persisted artifacts) before final sign-off.

## Phase 1: Extract and classify claims

Read the text carefully. Enumerate every distinct claim — do not bundle multiple assertions into one.

**Taxonomy:**
| Type | What it is | Treatment |
|------|-----------|-----------|
| **FACT** | Specific, verifiable: numbers, names, dates, events, attributions, statistics, causal claims | Search + verdict + edit proposal |
| **OPINION** | Subjective: judgments, predictions, interpretations, value statements | Flag only. No citation required. No edit. |
| **HEURISTIC** | Generalization: "usually," "often," "tends to," "generally," "most," "typically" | Flag as arguable. No hard verdict. |

Present the claim list to Sloane and ask if anything was missed before proceeding.

---

## Phase 2: Search evidence (parallel, multi-source)

Use multiple search strategies in parallel for each FACT claim. The goal is to get at least 2–3 independent signals per claim.

### Strategy A: WebSearch tool (primary)

Use the `WebSearch` tool directly for each FACT claim. This is the simplest and most reliable path.

### Strategy B: SearXNG + EXA helper script (supplementary)

Run the helper script to fire parallel HTTP searches via SearXNG and EXA. This provides additional sources beyond what WebSearch finds.

```bash
bws run -- uv run python .claude/skills/fact-checker/scripts/fact_check.py \
  --file path/to/document.md --json --verbose
```

The script handles SearXNG + EXA in parallel and returns structured JSON with sources per claim. It also uses Claude API for claim extraction and verdict generation — but when running from Claude Code, use the script primarily for its **search capability** (the `--json` output includes sources), and do your own assessment in Phase 3.

### Strategy C: Subagents for large documents (10+ FACT claims)

Spawn one subagent per claim (or batch of 2–3 claims), each using WebSearch + WebFetch to find evidence, then merge the results back. Dispatch them unnamed so each report returns inline.

This gives up to three independent search signals per claim:
1. WebSearch (Claude Code built-in)
2. SearXNG (via helper script, self-hosted broad web)
3. EXA neural (via helper script, semantic search for nuanced claims)

### When to use which

- **< 5 FACT claims**: WebSearch tool only (Strategy A). Fast and sufficient.
- **5–10 FACT claims**: WebSearch + helper script (A + B). Better recall.
- **10+ FACT claims**: All three (A + B + C). Subagents prevent serial bottleneck.

### Search provider reference (for manual use if needed):

```
SearXNG: GET {SEARXNG_URL}/search?q={query}&format=json&language=en&pageno=1
         SEARXNG_URL defaults to https://api.ec1c.com/api/v1/research/searxng
         (internal-api, CF Access pair required; 503 until the VPC binding lands — ETH-ixivw 3/3)
EXA:     POST https://api.exa.ai/search
         Headers: x-api-key: {EXA_API_KEY}
         Body: {"query": "...", "numResults": 6, "type": "neural", "useAutoprompt": true}
```

---

## Phase 3: Assess

For each FACT claim, assign:

| Verdict | Meaning |
|---------|---------|
| **CONFIRMED** | Multiple reliable sources directly support the claim |
| **REFUTED** | Sources contradict the claim |
| **MISLEADING** | Technically true but framing is selective or deceptive |
| **NUANCED** | Partially true with important caveats |
| **UNVERIFIED** | No reliable evidence found either way |

Confidence: **HIGH** (strong/consistent) · **MEDIUM** (partial/gaps) · **LOW** (thin/conflicting)

---

## Phase 4: Propose edits

For every FACT claim, propose one concrete edit. Check the brand voice skill before proposing any rewording:

```
Invoke: ecic-brand
Purpose: ensure proposed rewording matches ECIC's documented voice
```

**Edit type decision tree:**

```
CONFIRMED + good source URL available
  → add_link:     hyperlink the claim text to the best source
  → add_footnote: if the text already has footnotes OR the link would disrupt flow

REFUTED (HIGH/MEDIUM confidence)
  → reword:       correct the claim using best source evidence

REFUTED (LOW confidence) or MISLEADING or NUANCED
  → reword:       add a caveat inline, cite source
  → soften:       hedge the language if correction is uncertain

UNVERIFIED
  → soften:       change definitive language to hedged ("may be", "reportedly", "evidence suggests")
  → flag:         append [citation needed] if softening changes the meaning too much

CONFIRMED + no good source OR edit would be awkward
  → none
```

**Edit format rules:**
- Hyperlinks: `[claim text](https://source.url)` — link the minimum necessary span
- Footnotes: `claim text[^N]` with `[^N]: Full citation. Author, Publication, Date. URL.` at document end
- Rewording: replace the span verbatim; preserve surrounding sentence rhythm; do not over-hedge
- Softening: change e.g. "is the largest" → "is among the largest"; "always" → "typically"
- Do not change opinions or heuristics — only FACT claims get edits

---

## Phase 5: Interactive approval loop

Present each proposed edit one at a time. Wait for Sloane's decision before advancing.

**Format per edit:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Edit 1 of N | Claim [N] | REFUTED `HIGH` | reword
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

ORIGINAL:
  "[exact text span]"

PROPOSED:
  "[replacement text]"

FOOTNOTE (if applicable):
  [^N]: Full citation text.

ANALYSIS: One sentence summary of what the evidence says.
SOURCE: [Title](url)

→ Approve (A) · Modify (M) · Skip (S) · Done (Q)
```

**Handling responses:**
- **A / approve / yes**: Use the Edit tool to replace `original_text` with `proposed_text` in the source file. If `footnote_text` is present, append the footnote to the document. Confirm: "Edit applied." Then advance.
- **M / modify / [dictated change]**: Incorporate Sloane's modification into the proposed text. Show the revised version and wait for explicit approval before applying.
- **S / skip / no**: Note skipped, advance.
- **Q / done / stop**: End the loop, show a summary of what was applied vs. skipped.

**After all edits:**
Show a one-line summary:
```
Applied 4 edits (2 links, 1 footnote, 1 reword) · Skipped 2 · File saved: path/to/document.md
```

---

## Key rules

- **Never invent sources** — only cite URLs that appeared in actual search results
- **Never modify opinions or heuristics** — only FACT claims get edits
- **Do not apply any edit without explicit approval** — show first, apply only on A/approve
- **Preserve Sloane's voice** — rewording must not change tone; invoke the `ecic-brand` skill if uncertain
- **One edit per claim** — do not propose multiple alternatives; pick the best one
- If SearXNG answers 503 / is unreachable (or `CF_ACCESS_CLIENT_ID`/`SECRET` are absent), skip it and note it; WebSearch + EXA still run
- If `EXA_API_KEY` is missing, WebSearch + SearXNG still run; note EXA is unavailable
- If WebSearch is unavailable, fall back to the helper script (SearXNG + EXA) only

## Failure Modes to Watch

- Claim edits are applied without explicit approval.
- Sources are cited but not durably persisted/traceable for key claims.
- Heuristic/opinion content is incorrectly "fact-corrected" as if factual.

## Deprecated (do not use)

- Legacy command examples that depend on `doppler run -- ...`
- Tool references that assume non-Hermes-only editor primitives
- Hardcoded local filesystem paths in examples
