# Ethical Capital — Agent Skills

The skill library we publish for transparency: how we instruct our AI coding agents (Claude Code, Factory, pi, OpenCode) at [Ethical Capital](https://ethicic.com) — research, compliance, writing, and delivery discipline. Each skill is a directory with a `SKILL.md` (frontmatter: `name`, `description`); an agent loads it when a task matches.

This repository is a published snapshot, not the operating runbook for internal systems. Skills are authored and reviewed in our monorepo and published here by hand; the cross-environment setup contract lives in [docs/spikes/2026-10-18-skills-stack-design.md](docs/spikes/2026-10-18-skills-stack-design.md). This repository is canonical; [github.com/ethicalcapital/skills](https://github.com/ethicalcapital/skills) mirrors it.

## Skills

### Investment research and evidence

| Skill | Role |
|---|---|
| [research-notes](research-notes/SKILL.md) | Research-note structure and lifecycle |
| [research-reviewer](research-reviewer/SKILL.md) | Evidence integrity and note review |
| [pipeline-runner](pipeline-runner/SKILL.md) | Research-pipeline procedures (execution needs separate access) |
| [evidence-crawl](evidence-crawl/SKILL.md) | Gathering and preserving source material |
| [exclusions-manager](exclusions-manager/SKILL.md) | Exclusion-data lifecycle and quality |
| [writing-exclusions](writing-exclusions/SKILL.md) | Exclusion narratives and evidence |
| [investment-analysis](investment-analysis/SKILL.md) | Thesis structure and decision reasoning |
| [fact-checker](fact-checker/SKILL.md) | Claim-level verification |
| [compliance-analyst](compliance-analyst/SKILL.md) | RIA compliance-review procedures (not legal advice) |
| [r2-lake-sink-guardrails](r2-lake-sink-guardrails/SKILL.md) | Object-store access guardrails |

### Writing, judgment and brand

| Skill | Role |
|---|---|
| [buzz-saw-editor](buzz-saw-editor/SKILL.md) | Argument structure, compression, unsupported claims |
| [anti-racist-review](anti-racist-review/SKILL.md) | Identity-sensitive framing review |
| [veil-of-ignorance](veil-of-ignorance/SKILL.md) | Positional blind spots and who bears consequences |
| [ethical-editorial-gate](ethical-editorial-gate/SKILL.md) | Ethics gate for persuasive and identity-adjacent writing |
| [thought-partner](thought-partner/SKILL.md) | Structured reasoning through uncertain decisions |
| [ecic-brand](ecic-brand/SKILL.md) | Brand voice and standards |
| [serper-seo](serper-seo/SKILL.md) | Search-informed content research |
| [signup-flow-cro](signup-flow-cro/SKILL.md) | Signup-flow conversion analysis |

### Development discipline — vendored from [obra/superpowers](https://github.com/obra/superpowers) (MIT)

These fourteen workflow skills are adapted from Jesse Vincent's superpowers framework and keep its `superpowers:` cross-references.

| Skill | Role |
|---|---|
| [using-superpowers](using-superpowers/SKILL.md) | Discovering and applying workflow skills |
| [brainstorming](brainstorming/SKILL.md) | Requirements and design exploration |
| [writing-plans](writing-plans/SKILL.md) | Implementation planning |
| [executing-plans](executing-plans/SKILL.md) | Plan execution with checkpoints |
| [test-driven-development](test-driven-development/SKILL.md) | Red/green implementation discipline |
| [systematic-debugging](systematic-debugging/SKILL.md) | Reproduction and root-cause investigation |
| [verification-before-completion](verification-before-completion/SKILL.md) | Executed evidence before completion claims |
| [requesting-code-review](requesting-code-review/SKILL.md) | Review requests and context |
| [receiving-code-review](receiving-code-review/SKILL.md) | Verifying and addressing review findings |
| [subagent-driven-development](subagent-driven-development/SKILL.md) | Delegated implementation with review |
| [dispatching-parallel-agents](dispatching-parallel-agents/SKILL.md) | Independent parallel work |
| [using-git-worktrees](using-git-worktrees/SKILL.md) | Isolated development workspaces |
| [finishing-a-development-branch](finishing-a-development-branch/SKILL.md) | Integration and branch closeout |
| [writing-skills](writing-skills/SKILL.md) | Skill authoring and verification |

### Review, delivery and tracking

| Skill | Role |
|---|---|
| [code-review-lifecycle](code-review-lifecycle/SKILL.md) | End-to-end pull-request review flow |
| [execution-to-delivery](execution-to-delivery/SKILL.md) | Finishing the last 5% of delivery |
| [kata-issue-tracking](kata-issue-tracking/SKILL.md) | Organization issue-tracker workflow |

### Interfaces, documents and infrastructure

| Skill | Role |
|---|---|
| [accessibility-review](accessibility-review/SKILL.md) | Accessibility checks across human-facing output |
| [data-app-ui-review](data-app-ui-review/SKILL.md) | Data-application clarity and interaction review |
| [typeset-pdf](typeset-pdf/SKILL.md) | Document and PDF production |
| [cynical-schema-review](cynical-schema-review/SKILL.md) | Database design and migration-risk review |
| [auditing-claude-installation](auditing-claude-installation/SKILL.md) | Claude Code configuration audits |
| [rust-tui](rust-tui/SKILL.md) | Terminal UI patterns for our Rust clients |

## Vendored in our wider stack (not republished here)

Third-party skill families and tools we run machine-side, pointing at their upstreams:

- [obra/superpowers](https://github.com/obra/superpowers) — the workflow family above (MIT)
- [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill) — design-taste family (brandkit, design-taste-frontend, …)
- [tt-a1i/archify](https://github.com/tt-a1i/archify) — architecture diagrams as standalone HTML
- [pbakaus/impeccable](https://github.com/pbakaus/impeccable) and [@fradser/pi-impeccable](https://www.npmjs.com/package/@fradser/pi-impeccable) — interface design guidance and its Pi extension
- [backnotprop/plannotator](https://github.com/backnotprop/plannotator) — plan annotation family
- [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) and [vercel-labs/skills](https://github.com/vercel-labs/skills) — find-skills, web/writing guidelines
- RPIV workflow stages — [@juicesharp/rpiv-pi](https://www.npmjs.com/package/@juicesharp/rpiv-pi), [@juicesharp/rpiv-workflow](https://www.npmjs.com/package/@juicesharp/rpiv-workflow), [srvo/rpiv-praxis](https://github.com/srvo/rpiv-praxis)

## Install

Clone this repository (or its GitHub mirror), then copy a skill into your agent's skills directory:

```bash
cp -r buzz-saw-editor/ ~/.claude/skills/buzz-saw-editor/
```

Review a skill's license and supporting files before adopting it. A skill is instructions, not permission to access a system — supply tools and credentials separately.

## Ownership and privacy

One writer per skill: our monorepo authors and renders these trees, and this repository only publishes them. Ownership changes go through explicit reviewed handoffs. Private material — research notes, client data, personal collaboration learning — does not become publishable by being version-controlled; a public branch is publication.

## License

[CC BY-NC 4.0](LICENSE) for our own skills. Vendored skills retain their upstream licenses.
