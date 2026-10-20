---
name: execution-to-delivery
description: >
  Use when moving from plan to shipped artifact without dropping the final 5%.
  Trigger for multi-step implementation work, branch execution, and completion
  verification under delivery pressure.
version: 1.0.0
author: Ethical Capital
license: Proprietary
metadata:
  hermes:
    tags: [planning, execution, delivery, verification]
    related_skills: [writing-plans, executing-plans, verification-before-completion, finishing-a-development-branch, subagent-driven-development]
---

# Execution to Delivery

## Overview

This umbrella skill enforces the capture → structure → refine → deliver path for implementation work.

Leaf skills are sequenced to prevent “started but not shipped” outcomes.

## When to Use

- Any task with 3+ steps
- Any branch expected to produce working artifacts
- Any work where handoff/completion quality matters

## Workflow

1. `writing-plans`
   - Define objective, constraints, acceptance criteria.
2. `executing-plans`
   - Execute planned tasks in order with evidence.
3. `subagent-driven-development` (optional)
   - Parallelize independent subtasks when it reduces cycle time.
4. `verification-before-completion`
   - Confirm output matches requirements and runs.
5. `finishing-a-development-branch`
   - Cleanly package final state.

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
