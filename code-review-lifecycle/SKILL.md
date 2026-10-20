---
name: code-review-lifecycle
description: >
  Use when shipping code changes that require end-to-end review flow across request,
  receive, and schema skepticism. Trigger when opening PRs, responding to reviewer
  feedback, or validating DB-impacting changes before merge.
version: 1.0.0
author: Ethical Capital
license: Proprietary
metadata:
  hermes:
    tags: [code-review, pr, schema, quality-gates]
    related_skills: [requesting-code-review, receiving-code-review, cynical-schema-review, verification-before-completion, finishing-a-development-branch]
---

# Code Review Lifecycle

## Overview

This umbrella skill orchestrates the full review lifecycle so review quality does not depend on memory or mood.

Use the leaf skills in sequence:

1. `requesting-code-review` — shape the review request and acceptance criteria.
2. `cynical-schema-review` — apply schema-level skepticism before reviewer discovery.
3. `receiving-code-review` — process feedback without scope drift.
4. `verification-before-completion` — verify all acceptance criteria with evidence.
5. `finishing-a-development-branch` — close loop cleanly.

## When to Use

- Any PR intended for merge to shared branches
- Any change with database/schema implications
- Any change where reviewer time is scarce and precision matters

## Workflow

### Phase 1 — Pre-review hardening
- Run `requesting-code-review`
- Run `cynical-schema-review` if data shape, constraints, indexes, or migrations are touched
- Ensure tests and verification steps are reproducible

### Phase 2 — Active review handling
- Run `receiving-code-review`
- Convert reviewer comments into explicit actions
- Resolve comments with evidence, not assertions

### Phase 3 — Completion gate
- Run `verification-before-completion`
- Run `finishing-a-development-branch`
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
