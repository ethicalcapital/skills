---
name: kata-issue-tracking
description: Issue tracking via the kata CLI. MUST READ before starting work, issue/backlog triage, issue mutations, or next-task selection.
version: 1.0.0
---

# Issue Tracking

kata is the source of truth for work items (local daemon; `kata health` to confirm). `MEMORY_PROVIDER` must be unset or `kata` (`scripts/preflight/ensure-tracker-issue.sh` refuses anything else).

## Commands

| Operation | kata command |
|-----------|-------------|
| Create | `kata create` |
| Read | `kata show` |
| List | `kata list` |
| Edit | `kata edit` |
| Claim | `kata claim` |
| Close | `kata close` |
| Comment | `kata comment` |
| Label | `kata label` |
| Search | `kata search` |
| Ready | `kata ready` |
| Next | `kata next` |
| Relationships | `kata edit --blocks/--parent/--related` |
| Metadata | `kata meta` |

## Non-negotiables

- Track all work in the active tracker.
- Before starting any task, run the ready command.
- If no ready issue surfaces, create one immediately before implementation.
- Intake-first: if a user asks for new work, first find the matching issue or create one before implementation.
- File any discovered work >2 minutes with an issue create.
- Cover blockers and related follow-ups; link them with relationships.
- Before ending: persist issue state, then commit/push code/docs separately.
- Always use `--json` for programmatic access.

## Non-interactive commands only

Never run `create` without flags; it can hang waiting for input.

```bash
# Create
kata create "Title" --priority 2 --body "Context" --json
echo 'Body with `code`' | kata create "Title" --priority 2 --body-stdin --json

# Read
kata show <ref> --json
kata ready --json
kata next --json

# Mutate
kata assign <ref> <owner> --comment "Claiming: <why>"
kata close <ref> --done --message "What actually shipped" --evidence commit:<sha>
```

Priority: `0` highest, `4` lowest.

## Think → Create → Act

- Unknown bug: investigate briefly, then file discovered work with a `--related` link to the parent.
- Known feature: create the issue first with acceptance criteria, then work from `show`.
- Large goal: create an epic and immediately decompose into child tasks.
- Atomic task: one implementation step, clear done check.
- Dependencies: use `blocks` for hard blockers, `parent` for hierarchy, `related` for soft links; `ready` depends on blockers.

## Field discipline

- Title/body: stable problem statement; change rarely.
- Comments: session handoff (`COMPLETED`, `NEXT`, blockers).
- Status: claim immediately; never close on ticket text alone — repo/prod evidence required.
- Every `kata close` MUST pass `--evidence` (at least one of `commit:<sha>`, `pr:<url>`, `reviewed-paths:<paths>`, `test:<result>`) and a `--message` stating what shipped and where (1-2 sentences).
- After every close, confirm with a fresh `kata show`; a close with no landed state is not a close.
- A recorded note is NOT evidence the mutation happened. After ANY status mutation, confirm it landed with `show` before moving on.

### Releasing a stale claim (prune)

The status MUST be flipped in the same step that writes the release note, never note-only:

1. Reopen the ticket (flip out of the active/claimed state first).
2. Verify status is no longer claimed before writing any note; if it still reads claimed, stop and re-run step 1.
3. Only then append the release note.

## Swarm and subagent handoffs

- Always pass the active issue ID to peer agents/subagents. If none exists yet, create one first.
- Keep one parent issue as the coordination source; file child issues for substantial independent workstreams.
- Update the tracker at every ownership boundary (handoff, subagent return, blocked/unblocked, verification pass/fail).
- If delegated work uncovers new scope, create/link follow-up issues before continuing implementation.

## Search and triage

```bash
kata search "query terms" --hybrid --json    # semantic + lexical search
kata next --json                             # best next ready task
kata ready --json                            # all ready issues
```

Put durable context in issue bodies: source paths, evidence, decisions, constraints, acceptance checks. Use comments for chatter/logs. If a comment changes search meaning, summarize it into the body. No secrets.

## State persistence

The kata daemon persists state locally. `kata export` is an interchange artifact, never the source of truth.

## Metadata join

Every issue migrated from the pre-kata tracker has `metadata.beads_id` (its legacy `ETH-xxxx` id). To read it:

```bash
kata meta get <ref> beads_id
```

To find the kata issue from a legacy id:

```bash
kata list --meta beads_id=<id> --json
```

The `activity.timeline` payload `beads_id` field joins to the same metadata key. The pre-kata archive is read-only; never write to it.
