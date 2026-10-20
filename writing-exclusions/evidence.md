# Exclusion Evidence Bar

> Companion to SKILL.md in this directory. Rules for writing live in SKILL.md; what counts as proof lives here.

## Evidence Quality Standards (from 2026-03-31 audit)

These standards emerged from a full audit of 86 exclusion records, 16,259 evidence URLs, and validation of the top 22 SPY positions by weight. They encode what distinguishes a defensible exclusion from a liability.

### What counts as evidence

**Strong evidence (keeps exclusions alive):**
- Regulatory enforcement actions with case numbers (SEC, DOJ, FTC, OSHA, EPA, CFPB, state AGs)
- Court filings and settlements with dollar amounts and dates
- Peer-reviewed research (Science, Environmental Research Letters, One Earth)
- Named NGO investigations (HRW, Amnesty, AFSC Investigate, Who Profits, BHRRC, InfluenceMap)
- ViolationTracker aggregate penalty data with record counts
- Company disclosures that contradict company claims (internal docs vs. public statements)
- Government databases (OSHA inspection records, EPA ECHO, SEC EDGAR filings)

**Weak evidence (insufficient alone — needs corroboration):**
- Low benchmark/disclosure scores without documented incidents (e.g., KnowTheChain score of 8/100 with zero actual forced labor incidents — this got AVGO, 2.6% of SPY, downgraded to pipeline)
- Single employee criminal convictions where the company was the victim (e.g., Netflix VP bribery — company cooperated with prosecution)
- Advocacy group opinion pieces about reporting methodology inconsistencies (e.g., GEV emissions reporting — no SEC action, no actual fraud)
- De minimis fines for large companies ($157K VOC settlement for a $150B company is noise, not a pattern)
- Pending lawsuits about prospective harm that hasn't occurred yet

**Junk evidence (must be removed on sight):**
- Reddit threads, WordPress blogs, Medium posts, Substack newsletters
- Stock screener / investment blog articles ("Is this undervalued stock a hidden gem?")
- LinkedIn profile pages
- API endpoints not intended for human consumption
- Company's own anti-harm corporate statements cited as evidence OF harm (this is how KLAC got a conflict_zones exclusion from its own boilerplate)
- URLs from one company cited as evidence against a different company (Fairlife dairy case study cited against DRRX, a pharma company)
- Content farm statistics pages (gitnux.org, portersfiveforce.com)

### The pipeline hallucination problem

The research pipeline can generate plausible-sounding reason_detail text with fabricated or irrelevant evidence URLs. Signs of a hallucinated record:
- Reason_detail reads like a company description, not evidence of harm
- Evidence URLs are bare homepages with no path
- Evidence URLs point to completely unrelated content
- The reason_detail restates the company's own corporate policy as if it were an accusation
- Dollar amounts, citation numbers, or case references that can't be verified

**Every new pipeline-generated record should be treated as unverified until a human or research agent confirms the specific claims.** The pipeline is good at identifying *which* companies to investigate. It is unreliable at *documenting why*.

### Weight-aware audit methodology

Exclusions should be strongest where they cost the most. To audit efficiently:

```sql
-- Surface highest-cost, thinnest-rationale exclusions
WITH spy AS (
    SELECT holding_ticker, weight_pct, holding_company_id
    FROM research.fund_holdings WHERE fund_ticker = 'SPY' AND is_equity = true
),
exclusion_counts AS (
    SELECT company_id, COUNT(*) as reason_count
    FROM exclusions.active_exclusions
    WHERE valid_to IS NULL AND is_approved = true AND disposition = 'excluded'
    GROUP BY company_id
)
SELECT s.holding_ticker, s.weight_pct, ec.reason_count
FROM spy s JOIN exclusion_counts ec ON ec.company_id = s.holding_company_id
WHERE ec.reason_count <= 2
ORDER BY s.weight_pct DESC;
```

A baseless exclusion on a 2.6% SPY position costs more portfolio flexibility than a well-sourced exclusion on a 0.01% position. When auditing, start from the top of this list. The 2026-03-31 audit freed 4.1% of SPY weight (from 92.15% to 88.01% excluded) by removing 5 weak/baseless exclusions while simultaneously strengthening evidence on positions that stayed.

### Category accuracy matters

Miscategorized exclusions undermine credibility even when the underlying concern is real:
- U.S. immigration enforcement is NOT a "conflict zone" (CRM was reclassified to `border_enforcement`)
- A facility inside the Green Line is NOT "occupied territory" (INTC was reclassified to `military_contracting`)
- An NGO's opinion about emissions reporting is NOT "financial misconduct" (GEV was downgraded)
- FDA clinical holds during normal drug development are NOT "regulatory failures" (SLRX was retired)
- A company's own anti-conflict statement is NOT evidence of conflict zone involvement (KLAC was retired)

Before writing a record, read the taxonomy description for both the target code AND adjacent codes. If the evidence fits better next door, put it there.

### Dual-use technology

Some companies (Thermo Fisher, Intel, KLA) make products with both civilian and military applications. The test is not whether the product *could* be used for harm — it's whether the company has:
1. A dedicated defense/military division or marketing channel
2. Direct contracts with military/intelligence/border enforcement agencies
3. Knowledge that products are being used for surveillance, weapons manufacturing, or repression

Thermo Fisher has all three (defense applications page, $120M FBI/CODIS contract, documented Uyghur DNA surveillance). A semiconductor company whose chips end up in military systems through third-party integration is a weaker case unless there are direct military contracts.

