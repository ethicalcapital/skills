---
name: code-review-lifecycle
description: >
  Use when shipping code changes that require end-to-end review flow across request,
  receive, and schema skepticism. Trigger when opening PRs, responding to reviewer
  feedback, or validating DB-impacting changes before merge.
version: 1.1.0
author: Ethical Capital
license: Proprietary
metadata:
  hermes:
    tags: [code-review, pr, schema, quality-gates]
    related_skills: [cynical-schema-review]
---

# Code Review Lifecycle

## Overview

This umbrella skill orchestrates the full review lifecycle so review quality does not depend on memory or mood.

Run the phases in sequence; `cynical-schema-review` is the one leaf skill this depends on.

1. **Shape the request** — state the acceptance criteria and what you want reviewed.
2. `cynical-schema-review` — apply schema-level skepticism before reviewer discovery.
3. **Process feedback** — turn comments into explicit actions without scope drift.
4. **Verify** — prove each acceptance criterion with executed evidence.
5. **Close the loop** — leave the branch merge-ready and operationally clean.

## When to Use

- Any PR intended for merge to shared branches
- Any change with database/schema implications
- Any change where reviewer time is scarce and precision matters

## Workflow

### Phase 1 — Pre-review hardening
- Write the acceptance criteria into the PR body
- Run `cynical-schema-review` if data shape, constraints, indexes, or migrations are touched
- Ensure tests and verification steps are reproducible

### Phase 2 — Active review handling
- Convert reviewer comments into explicit actions
- Resolve comments with evidence, not assertions

### Phase 3 — Completion gate
- Verify every acceptance criterion with executed evidence
- Confirm branch is merge-ready and operationally clean

## Common Pitfalls

1. Treating reviewer comments as suggestions instead of requirements.
2. Missing schema edge cases because review started at UI/app layer.
3. Replying with intent (“should be fixed”) instead of verification evidence.

## Verification Checklist

- [ ] Review request includes explicit acceptance criteria
- [ ] Schema skepticism pass completed when relevant
- [ ] Reviewer feedback resolved with concrete evidence
- [ ] Final verification pass completed before merge
- [ ] Branch completion checklist run
