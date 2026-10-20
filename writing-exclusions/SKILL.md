---
name: writing-exclusions
description: Use when writing reason_detail narratives for exclusions.active_exclusions records, or when composing exclusion prose for the website at /exclusions/{ticker}/. Triggers on "write exclusion entry", "write reason_detail", "draft exclusion narrative", or any request to document why a company is excluded.
---

# Writing Exclusion Entries

`reason_detail` is the public-facing prose in `exclusions.active_exclusions`. It renders on ethicic.com/exclusions/{ticker}/ and is the authoritative explanation of why a company is excluded. One record per `sub_category_code` — a company with three exclusion reasons gets three separate narratives.

## The Voice

Third-person factual narrative. You are sharing a finding, not writing an oo-ed. If the evidence is damning, just state the evidence.

## Structure

1-3 paragraphs of plain prose. No markdown headers, no bullet points, no tables, no "Excluded under X.X (policy name)" formulae. Just sentences.

**Paragraph 1**: What the company does that triggers this specific exclusion code. Name the business activity, its scale, and its centrality to the company. Revenue percentages, customer counts, asset values.

**Paragraph 2** (if needed): The specific conduct, incident, or pattern. Dates, dollar amounts, regulatory bodies, case numbers, settlement terms. Named sources (ViolationTracker, InfluenceMap, BHRRC, SEC, DOJ).

**Paragraph 3** (if needed): Context that sharpens or complicates the picture — transition plans that are real vs. performative, mitigating facts that matter, or the absence of commitments that should exist.

## What Good Looks Like

**CenterPoint Energy — political_influence:**
> CenterPoint sits on the board of the American Gas Association, which opposes building electrification policy. The company is a member of ALEC and Consumer Energy Alliance. It supported Texas SB 1261, which blocks municipal greenhouse gas regulation, and Indiana PL 180, which blocks fossil gas bans.
>
> In 2021, a CenterPoint lobbyist ghostwrote third-party comments submitted to Minnesota regulators supporting gas appliance rebates. The company acknowledged providing "guidance and sample letters." InfluenceMap assesses CenterPoint as taking "mostly negative positions on the energy transition" and "strategically advocating for fossil gas infrastructure."

**NiSource — natural_gas:**
> On September 13, 2018, over-pressurization of Columbia Gas of Massachusetts pipelines caused approximately 80 homes in Lawrence, Andover, and North Andover to explode or catch fire simultaneously. One person was killed and at least 25 were injured. The Massachusetts Attorney General reached a $56 million settlement. Residents and businesses received $143 million. Municipalities received $83 million. NiSource subsequently sold its Massachusetts gas operations to Eversource for $1.1 billion.
>
> The incident was the worst natural gas utility disaster in modern U.S. history. NiSource continues to operate Columbia Gas in six other states, distributing natural gas to 3.5 million customers with no announced phase-out plan.

**Seaboard Corporation — animal_exploitation:**
> Seaboard Corporation is a major vertically integrated pork producer, operating large-scale hog confinement facilities and processing plants primarily through its Seaboard Foods subsidiary. In 2003, workers at a Seaboard Farms facility in Oklahoma were charged with felony animal cruelty after a PETA undercover investigation documented systemic abuse of pigs. Seaboard Foods processes approximately 20,000 hogs per day at its Guymon, Oklahoma plant, making it one of the largest pork processing operations in the United States.

## Rules

1. **One record = one sub_category_code.** Don't cover multiple exclusion reasons in a single entry. CenterPoint has separate entries for `community_harm`, `fossil_fuels_midstream`, and `political_influence` — each tells its own story. Before writing, check the taxonomy description for the code you're writing — it tells you exactly what belongs here vs. next door. Read the descriptions for adjacent codes too.

2. **Route off-code evidence to review_note.** If you find evidence that doesn't fit the current `sub_category_code` but is real and material, don't drop it — write it into the record's `review_note` field with a date stamp, source attribution, and a note about which code it might belong under. Example: `[2026-03-16: AFSC documents 53 BOP contracts ($9M, 2008-2021). Does not fit military_contracting — may warrant separate record under private_prisons.]` This preserves the evidence for CIO triage without contaminating the narrative.

3. **Lead with scale and specificity.** Not "the company has fossil fuel exposure" — instead "derives approximately 47% of revenue from natural gas distribution, serving over 4 million gas customers."

4. **Numbers baked into sentences, not listed.** Not "Revenue: $19B. Customers: 4M." Instead: "The company has committed $19 billion to gas infrastructure investment through 2035."

5. **Source attribution in the prose.** "InfluenceMap assesses..." "ViolationTracker documents..." "BHRRC 2025 data links..." "The SEC order (Administrative Proceeding File No. 3-14380) documented..."

6. **Let facts editorialize.** Not "the company is irresponsibly expanding fossil infrastructure." Instead: "The company has committed $19 billion to gas infrastructure through 2035 — not maintaining legacy systems, but actively expanding the network." The em-dash does the work.

7. **Name the absence.** "CenterPoint has no Science Based Targets initiative commitment. Its self-set net-zero target excludes Scope 3 emissions from sold gas, which is the dominant emission source for a gas distribution utility." Stating what doesn't exist is as important as stating what does.

8. **Acknowledge real complexity.** "The natural gas exposure is real but directionally shrinking under binding state law." "Gas distribution is a regulated utility obligation, not a strategic fossil fuel bet." Don't flatten nuance — the credibility of the exclusion depends on it.

9. **When a record should be lifted, say so.** "Six targeted searches found no evidence that Federal Realty Investment Trust leases properties to ICE... This record has been lifted." "The original fossil fuel exclusion predates this divestiture and is factually stale."

10. **Consistent tense.** Present tense for ongoing business activities ("Caterpillar operates a Government & Defense division"). Past tense for specific events ("settled in 2022 for $740 million"). Never mix tenses for the same timeframe — "holds 53 contracts between 2008 and 2021" is wrong because the contracts are historical.

## CIO Calibration Standards

These judgment calls encode how the CIO evaluates severity, materiality, and recency. Apply them when writing narratives.

**Escalating threshold (conduct codes):** One incident is alarming and merits observation. Two independent incidents are very concerning. Three or more isolated incidents with unrelated plaintiffs/sources showing the same fact pattern is dispositive of a systemic problem. This applies to: discrimination, harassment, working_conditions, anti_union, regulatory_failures, and OSHA violations. Note: OSHA penalties are often capped by statute (~$16K max per violation) — small dollar amounts do not mean small violations. It's the pattern.

**Recency and reform:** Companies can heal. A company with a terrible historical record but genuine, sustained reform should be reassessed. Ørsted went from the dirtiest energy company in Europe to the cleanest. State the historical record AND the current trajectory. Exclusions should reflect the present reality, not permanent punishment.

**Military contracting materiality:** Not every company that sells something to the military is a defense contractor. Exclusion thresholds: (1) specialized products/marketing targeting military customers, (2) military revenue >15% of total, or (3) purpose-built weapons systems. Dual-use commodity products (semiconductors, extension cords, test equipment) purchased by militaries are NOT grounds. Supporting military personnel as human beings (healthcare, insurance, worker protections) is NOT grounds. The line is between the war-industrial complex and incidental service to people who work in it.

**Emissions are peer-relative:** A retailer with an embarrassing carbon footprint relative to apparel peers may be excluded even if its absolute emissions are small. The questions: How does this company compare to its sector? Does leadership treat emissions governance as part of their job?

**Conflict zone affirmative defense:** For a company operating in a conflict zone to avoid exclusion, there must be an affirmative argument that the civilian population would be materially worse off without the company's service (e.g., the only source of heat or power). Partnering with a junta to run a mine gets no such defense.

**PFAS routing:** PFAS in pesticide formulations → `pesticides`. PFAS from non-pesticide products (Teflon, firefighting foam, food packaging) → `environmental_damage` or a future harmful products code, not `pesticides`.

## Evidence bar

Full tiers, hallucination guard, and audit method live in evidence.md beside this file. The rule: every code needs ≥1 independent source; tiers stack; official-responses-only fails (2026-03-31 audit: 44% fail).

## Anti-patterns

| Don't write | Write instead |
|-------------|---------------|
| "Excluded under 2.3 (Addictive Products) for tobacco manufacturing." | "Philip Morris International derives approximately 70% of its $35.2 billion revenue from combustible cigarette manufacturing across 180 markets." |
| "The company has environmental concerns." | "PPL has accumulated $213.7 million in environmental penalties across 43 enforcement records documented by ViolationTracker." |
| "There are labor violations in the supply chain." | "BHRRC 2025 data links Meta to the highest number of global migrant and gig worker abuse cases among major tech companies, concentrated in content moderation workforce." |
| "Significant regulatory issues." | "The Massachusetts Attorney General reached a $56 million settlement. Residents and businesses received $143 million." |
| "Industry-leading emissions" | "LG&E and KU operate a generation fleet that is approximately 79% coal-fired, 20% natural gas, and 1% renewable — among the most carbon-intensive in the U.S. utility sector." |

## Database Fields

Each entry updates `exclusions.active_exclusions`:

```sql
UPDATE exclusions.active_exclusions SET
  reason_detail = 'Your narrative here',
  review_status = 'needs_cio_review'
WHERE id = '<uuid>';  -- always target by row ID
```

- `reason_detail`: The prose narrative (plain text, no markdown)
- `review_status`: Set to `needs_cio_review` for CIO sign-off before going live
- `evidence_url` / `evidence_urls`: Direct URL to primary source document
- `evidence_citation`: Formal citation string ("ViolationTracker Global, accessed 2026-03-10")

All INSERT statements must use `ON CONFLICT DO NOTHING` or `ON CONFLICT ... DO UPDATE`.

---

## Taxonomy Reference

See taxonomy.md beside this file (auto-generated 2026-03-31; stale if older than 12 months; refresh via `bws run -- uv run data-pipelines/monthly/refresh_exclusion_taxonomy.py`). Read the target code's row and adjacent codes before writing.

## Deprecated (do not use)

- `doppler run -- ...` refresh commands — this repo uses `bws run --`
