---
name: execution-to-delivery
description: >
  Use when moving from plan to shipped artifact without dropping the final 5%.
  Trigger for multi-step implementation work, branch execution, and completion
  verification under delivery pressure.
version: 1.1.0
author: Ethical Capital
license: Proprietary
metadata:
  hermes:
    tags: [planning, execution, delivery, verification]
---

# Execution to Delivery

## Overview

This umbrella skill enforces the capture → structure → refine → deliver path for implementation work.

Leaf steps are sequenced to prevent “started but not shipped” outcomes.

## When to Use

- Any task with 3+ steps
- Any branch expected to produce working artifacts
- Any work where handoff/completion quality matters

## Workflow

1. **Plan.** State the objective, constraints and acceptance criteria on a Forgejo issue before touching code.
2. **Execute.** Work the planned tasks in order, keeping evidence for each one.
3. **Parallelize** (optional). Split independent subtasks when it shortens the cycle; dispatch subagents unnamed so their reports return inline.
4. **Verify.** Confirm the output matches the requirements and actually runs.
5. **Deliver.** Package the final state: rebase on `dev`, push the branch, open the PR.

## Completion Rule

Do not mark complete until the requested artifact is actually built/run/verified.

## Common Pitfalls

1. Stopping at “plan written” or “code drafted.”
2. Conflating partial execution with completion.
3. Skipping final verification because prior steps seemed correct.

## Verification Checklist

- [ ] Plan includes explicit acceptance criteria
- [ ] Execution evidence exists for each planned step
- [ ] Artifact was run or otherwise materially validated
- [ ] Final branch hygiene and completion steps executed
