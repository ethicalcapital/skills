---
name: serper-seo
description: "Use when you need to research keyword opportunities, analyze SERP competition, find long-tail queries, or inform content framing decisions using real search data. Requires SERPER_API_KEY in BWS."
---

# Serper SEO Research

Use Serper's Google Search API to research keyword demand, competition, and content opportunities for the international retirement guide (or any SEO task).

## API Access

`SERPER_API_KEY` lives in BWS. `bws run --` injects it into the
environment of the command it wraps — never fetch the value into a shell
variable, and never interpolate it in an outer shell (see
`~/.claude/rules/secret-handling.md`). Because the header has to be built from
`$SERPER_API_KEY`, write the request to a script file and let *that* read the
env, rather than inlining it (`~/.claude/rules/claude-code-bash-quoting.md`):

```bash
cat > /tmp/serper.sh <<'EOF'
#!/usr/bin/env bash
# $1 = endpoint (search|news), $2 = query, $3 = num results
curl -s -X POST "https://google.serper.dev/$1" \
  -H "X-API-KEY: $SERPER_API_KEY" \
  -H "Content-Type: application/json" \
  --data "$(jq -nc --arg q "$2" --argjson n "${3:-10}" '{q: $q, num: $n}')"
EOF
chmod +x /tmp/serper.sh

bws run -- /tmp/serper.sh search "your query here" 10 | jq .
bws run -- /tmp/serper.sh news   "your query here" 10 | jq .
```

## Research Workflow for Content Strategy

### Step 1: Seed Keyword Research

Search the core framing questions to understand intent and competition:

```bash
cat > /tmp/serper-seed.sh <<'EOF'
#!/usr/bin/env bash
for query in \
  "should I leave America" \
  "moving abroad as a Black American" \
  "LGBTQ expat retire abroad" \
  "trans people moving abroad" \
  "best countries for Black Americans" \
  "retiring abroad LGBTQ" \
  "international retirement guide" \
  "best places to retire abroad safety" \
  "leaving America 2025" \
  "expatriate guide marginalized Americans"; do
  echo "=== $query ==="
  curl -s -X POST https://google.serper.dev/search \
    -H "X-API-KEY: $SERPER_API_KEY" \
    -H "Content-Type: application/json" \
    --data "$(jq -nc --arg q "$query" '{q: $q, num: 5}')" | jq '.organic[].title'
  echo ""
done
EOF
chmod +x /tmp/serper-seed.sh

bws run -- /tmp/serper-seed.sh
```

### Step 2: Analyze Top Results

For each top-ranking page, note:
- **Title structure**: what framing do they use?
- **URL slug**: what keyword do they target?
- **Content depth**: comprehensive guide or listicle?
- **Site authority**: niche blog, major publisher, government?
- **Missing angle**: what do they not cover (LGBTQ+, trans, Black expat, disability)?

### Step 3: "People Also Ask" Mining

```bash
bws run -- /tmp/serper.sh search "retire abroad as American 2025" 10 \
  | jq '.peopleAlsoAsk[].question'
```

### Step 4: Related Searches

```bash
bws run -- /tmp/serper.sh search "best countries to move to from USA" 10 \
  | jq '.relatedSearches[].query'
```

## Key Data Points to Extract

For each query, capture:
- `organicResults[].title` — page title (framing)
- `organicResults[].link` — URL (slug keywords)
- `organicResults[].snippet` — content summary
- `peopleAlsoAsk[].question` — intent signals
- `relatedSearches[].query` — adjacent keywords
- `answerBox` — if present, what answer Google surfaced (AI SEO target)

## Interpreting Results for Content Decisions

### URL Slug Decision
- If top results use "retire-abroad" → keep `/international-retirement/`
- If top results use "expat-guide" or "move-abroad" → consider adding those as alternate entry points
- If "leaving-america" or "move-abroad" dominates → add a redirecting entry point page

### Title/H1 Framing
- Look for shared patterns: "Best Countries for X", "Complete Guide to X", "X: What You Need to Know"
- Identify the specific angle not covered by competitors (LGBTQ+, trans, Black expat, disability access)
- The "should I leave" framing is conversational — check if it beats "best countries to retire" in search volume

### Long-Tail Opportunities
- City-specific: "retiring in Lisbon as Black American" → direct destination page optimization
- Identity-specific: "LGBTQ friendly countries retirement" → Safety by Identity page optimization
- FAQ format: "People Also Ask" questions → add FAQ section to pages

## Response Format

After running research, deliver:
1. **Top-ranking competition analysis** — who ranks, what angle they take, gaps they miss
2. **Recommended URL structure** — based on dominant keyword patterns
3. **Recommended title/H1** — for the guide index page
4. **Top 10 long-tail opportunities** — city + identity keyword combos
5. **FAQ questions to add** — from People Also Ask, formatted for schema markup

## Deprecated (do not use)

- `doppler secrets get SERPER_API_KEY --plain` — the key is in BWS, and fetching a
  secret to stdout is forbidden regardless (`~/.claude/rules/secret-handling.md`).
  Let `bws run --` inject it and have the script read `$SERPER_API_KEY`.
