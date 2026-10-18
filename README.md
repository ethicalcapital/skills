# Ethical Capital — Agent Skills & Tooling

A catalog of the skill library published by [Ethical Capital](https://ethicic.com), the upstream tools that complement it, and the setup boundaries that keep them from becoming competing copies.

**Start here:** [the stack](#the-stack) · [skill catalog](#skill-catalog) · [installation and setup](#installation-and-setup) · [research-skill ownership](#research-skills-version-controlled-and-centrally-owned) · [provisioning design](docs/spikes/2026-10-18-skills-stack-design.md).

**Status:** this repository contains published skill files, not the entire live agent configuration. The cross-environment setup contract is documented below; installer convergence and live Hermes activation are not complete. Do not treat this published snapshot as the current operating runbook for internal systems.

## The stack

A portable skill, a native extension, a workflow engine and a service connector are different things. Installing one does not install the others or grant access to their services.

| Component | Purpose | Distribution and compatibility |
|---|---|---|
| **This skill library** | Research, writing, review, delivery and infrastructure instructions | Selected `SKILL.md` trees, including their supporting files. Host-specific commands and dependencies still need review. |
| **RPIV skills and Pi integration** | Explicit research, design, planning, implementation and validation stages | [`@juicesharp/rpiv-pi`](https://www.npmjs.com/package/@juicesharp/rpiv-pi). Keep the native package rather than copying its skill library into another managed tree. |
| **RPIV workflow engine** | Executes workflow definitions with stage dependencies and run state | [`@juicesharp/rpiv-workflow`](https://www.npmjs.com/package/@juicesharp/rpiv-workflow). Engine configuration and workflow packs are separate from the skill files they reference. |
| **RPIV Praxis** | Headless CLI/MCP hosting of workflows through the Claude Agent SDK | [`srvo/rpiv-praxis`](https://github.com/srvo/rpiv-praxis). This is workflow tooling, **not** the Praxis product application. Installing it does not start a run. |
| **Impeccable for Pi** | On-demand interface design, accessibility, typography, motion and critique guidance | [`@fradser/pi-impeccable`](https://www.npmjs.com/package/@fradser/pi-impeccable), a native Pi extension exposing `/impeccable` and `impeccable_load`. It is not a portable skill directory. |
| **Impeccable for other hosts** | The upstream design guidance, through supported host integrations | [Impeccable upstream](https://github.com/pbakaus/impeccable). Select a compatible integration; a Pi extension cannot simply be installed in Hermes or Claude Code. |
| **Other portable libraries** | Architecture diagrams, frontend design and framework-specific practices | Upstream packages such as [Archify](https://github.com/tt-a1i/archify), [taste-skill](https://github.com/Leonxlnx/taste-skill) and [Vercel agent skills](https://github.com/vercel-labs/agent-skills). Retain their own provenance, licenses and update paths. |
| **Browser, issue and service tools** | Browser actions, work tracking, retrieval and authenticated service access | Separate native packages, CLIs and MCP configuration. TinyFish skill instructions do not supply a browser account; a tracking skill does not install Kata. |

The Impeccable Pi package inspected for this setup is **0.1.1**. It is pre-1.0: review compatibility and changes before promoting an update. Its package manifest and README, not a similarly named design-skill collection, establish the integration above.[^impeccable]

RPIV stages remain explicitly invoked. Installing packages, rendering configuration or bootstrapping a machine must not launch a workflow, spend an LLM budget, or execute a research pipeline.

## Skill catalog

These links describe the **files actually published in this checkout**. Some operational instructions are historical. Publication here does not certify that an instruction matches current internal services, regulation or approval policy; use the maintained organization sources for internal execution.

### Investment research and evidence

| Skill | Role |
|---|---|
| [research-notes](research-notes/SKILL.md) | Research-note structure and lifecycle |
| [research-reviewer](research-reviewer/SKILL.md) | Evidence integrity, freshness and note review |
| [pipeline-runner](pipeline-runner/SKILL.md) | Research-processing procedures; execution requires separate access and authorization |
| [evidence-crawl](evidence-crawl/SKILL.md) | Gathering and preserving source material |
| [exclusions-manager](exclusions-manager/SKILL.md) | Exclusion-data lifecycle and quality |
| [writing-exclusions](writing-exclusions/SKILL.md) | Exclusion narratives and supporting evidence |
| [investment-analysis](investment-analysis/SKILL.md) | Thesis structure, assumptions and decision reasoning |
| [fact-checker](fact-checker/SKILL.md) | Claim-level verification |
| [compliance-analyst](compliance-analyst/SKILL.md) | Compliance-review procedures, not a substitute for current legal advice |
| [r2-lake-sink-guardrails](r2-lake-sink-guardrails/SKILL.md) | Object-store access and archive guardrails |

### Writing, judgment and brand

| Skill | Role |
|---|---|
| [buzz-saw-editor](buzz-saw-editor/SKILL.md) | Argument structure, compression and unsupported claims |
| [anti-racist-review](anti-racist-review/SKILL.md) | Review of identity-sensitive framing and assumptions |
| [veil-of-ignorance](veil-of-ignorance/SKILL.md) | Stakeholder consequences and positional blind spots |
| [thought-partner](thought-partner/SKILL.md) | Structured reasoning through uncertain decisions |
| [ecic-brand](ecic-brand/SKILL.md) | Ethical Capital voice and brand standards |
| [serper-seo](serper-seo/SKILL.md) | Search-informed content research |
| [signup-flow-cro](signup-flow-cro/SKILL.md) | Signup-flow analysis |

### Development discipline and delivery

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

These procedural skills are not aliases for RPIV stages. A workflow that calls a general planning skill has not necessarily executed the RPIV planning contract.

### Interfaces, documents and infrastructure

| Skill | Role |
|---|---|
| [accessibility-review](accessibility-review/SKILL.md) | Accessibility checks across human-facing outputs |
| [data-app-ui-review](data-app-ui-review/SKILL.md) | Data-application clarity and interaction review |
| [typeset-pdf](typeset-pdf/SKILL.md) | Document and PDF production procedures |
| [cynical-schema-review](cynical-schema-review/SKILL.md) | Database design and migration-risk review |
| [deploy-worker](deploy-worker/SKILL.md) | Worker release procedures |
| [workers-best-practices](workers-best-practices/SKILL.md) | Worker implementation patterns |
| [workers-observability](workers-observability/SKILL.md) | Worker health and diagnostics |
| [auditing-claude-installation](auditing-claude-installation/SKILL.md) | Claude Code configuration audits |

### Historical material

[beads-issue-tracking](beads-issue-tracking/SKILL.md) remains in this published snapshot. It is **not** the current organization tracking procedure: Kata is the authoritative tracker. Do not include the historical tracking skill in a current setup profile.

## Research skills: version-controlled and centrally owned

The thing to centralize is the **reviewed instruction source**, not a pile of independently editable installations.

- **One writer per skill.** Organization-authored research skills currently belong in `machine/agents/skills/` in monocloud. Its renderer produces the committed harness copies. This public repository is a publication surface, not an automatic second authoring source for those same names.
- **Ownership changes are explicit.** A skill may become upstream-owned only through a reviewed handoff. A matching filename or matching bytes does not authorize replacing an existing local tree.
- **Consumers use reviewed revisions.** Record the full source revision and complete-tree digest, then install through the appropriate owner/adapter. Keep supporting references, scripts, executable modes and license information with the skill.
- **Distribution is one-way.** Reviewed source → selected deployment → verification. Hermes-authored proposals follow their separately approved publication path; installed central research skills must not silently become Hermes-authored upstream content.
- **Private is not public.** Research notes, client information, internal source material and personal collaboration learning do not become publishable because the instructions are version controlled. Public branches are publication too.

The research profile should identify its selected skill set, source revision, owner and verification result. It must distinguish what was configured, downloaded, installed and actually loaded. Product applications consuming those profiles are a later phase, not part of this repository refresh.

## Installation and setup

### For someone adopting a skill

1. Review the skill's license, instructions and every supporting file.
2. Select a reviewed revision and keep a dedicated source checkout. Do not pull over a dirty working library.
3. Use your host's supported skill path or selected-directory link. Do not overwrite an existing installation, install the whole catalog blindly, or combine central copies with another package exporting the same skill name.
4. Supply required tools and credentials separately. A skill is instructions, not permission to access a system.
5. Check the host's discovered resources and diagnostics. File presence alone is not proof that the right version loaded.

For Pi, `~/.agents/skills` is natively discoverable. Do not also install a separate copy of the same shared skill under `~/.pi/agent/skills`. Other hosts need their supported adapter; native extensions remain native packages.[^pi]

### For organization maintainers

Provisioning stays with the existing owners rather than a new universal sync script:

| Environment | Existing entrypoint / owner | Current boundary |
|---|---|---|
| Local development machines | monocloud `machine/bootstrap.sh`, `machine/agents/` and canonical dotfiles | Bootstrap invokes the RPIV Praxis installer and home renderer. Rendering package declarations is not proof of package installation. |
| Cloud coding agents | monocloud `scripts/setup/cloud-agent.sh`; Cursor/Claude cloud shims forward to it | Current bootstrap clones the legacy GitHub skills source but does not converge the full agent stack. This needs explicit setup wiring and tests. |
| Hermes | Dedicated Hermes provisioning and mounted skill library | Install compatible portable profiles, not Pi extensions. Preserve authored content and require ownership-aware updates; do not use a replacement-only refresher for research skills. |
| Product application runtimes | Later, explicit consumer integration | No application, authentication, service or research-pipeline behavior changes are authorized by this catalog. |

**Chosen setup direction:** declare Impeccable alongside the existing native package declarations; connect selected resource installation to local/cloud setup; distribute approved research profiles to Hermes without replacing private learning. Verify each environment separately. See the [provisioning design and acceptance criteria](docs/spikes/2026-10-18-skills-stack-design.md) before implementing these changes.

Canonical configuration and operational runbooks live in the organization repositories, which require organization access. The paths above identify their owners; they are not scripts shipped by this repository. Existing GitHub consumers have not automatically switched to Forgejo merely because the Forgejo repository is writable.

## Maintenance and review rules

- Keep the catalog's **published**, **configured**, **installed** and **verified** states distinct. Do not retain stale plugin/tool counts as promises about every environment.
- Prefer maintained upstream packages. Review native code, licenses and host compatibility before installation; flag pre-1.0 dependencies.
- Keep credentials out of skill files, Git URLs, examples and logs. Existing organization CLI workflows use runtime Bitwarden injection, not embedded secrets.
- Verify both a clean install and refusal paths: wrong revision, unavailable scanner, conflicting names, local edits, missing dependencies and failed retries.
- Keep local experiments separate from centrally owned declarations. A setup command must not silently update all experimental packages or change provider credentials.
- Publish only reviewed material. Secret scanning does not establish privacy, provenance, licensing or permission to share.

## License

The material in this repository is licensed under [CC BY-NC 4.0](LICENSE): attribution is required, changes must be indicated, and commercial use is not permitted under that license. Upstream packages retain their own licenses; listing them here does not relicense them.

[^impeccable]: [Pi integration source](https://github.com/FradSer/pi-packages). The inspected `@fradser/pi-impeccable` package manifest declares version 0.1.1, a Pi extension entrypoint, MIT package licensing and Node.js ≥22.18.0; its README describes the upstream Apache-2.0 adaptations and bundled notices. This is an inspected-version statement, not a claim about the latest release.
[^pi]: [Pi skills documentation](https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/docs/skills.md) and [Pi packages documentation](https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/docs/packages.md). Review the documentation matching the host version before adopting an integration.
