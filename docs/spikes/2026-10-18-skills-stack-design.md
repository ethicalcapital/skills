# Agent skills stack and provisioning design

Tracking: kata `ryxa`, child of `d577`; related deployment issue `9d91`.

**Status:** the written contract and installer/configuration implementation were explicitly approved on 2026-10-18. Source implementation and review are in progress in the canonical setup repository; live activation and product-runtime integration remain deferred. Approval is not evidence that the stack has been deployed.

## Goal and approved scope

Make the public skills README the front door to a comprehensible, centrally maintained agent stack. Connect actual resource installation and verification to the existing local and cloud setup entrypoints. Include Hermes as a consumer of compatible, reviewed skill profiles.

Research skills must be version controlled and centrally owned. This concerns reusable instructions and their supporting assets—not publishing research notes, client information, proprietary evidence or personal collaboration history.

Product application runtimes are **later**. This work does not change the Praxis application's behavior, permissions, APIs, authentication, research execution or deployment.

## Alternatives considered

1. **README only:** quickest and useful for discovery, but leaves a freshly bootstrapped cloud agent without the documented stack.
2. **One universal vendored skill tree:** makes initial copying simple but mixes native extensions with portable instructions, loses upstream update ownership, creates name collisions and can overwrite local learning.
3. **Chosen: one catalog, existing owners, explicit consumer profiles.** Keep upstream native packages native; keep organizational authoring in canonical sources; reuse existing renderer, dotfiles and deployment entrypoints. Add only the glue and checks needed to make selected profiles converge.

No new skills service, bidirectional sync daemon or competing renderer is needed.

## Evidence and current gaps

The inspected sources establish the following; these are not live end-to-end acceptance results:

| Area | Current source / mechanism | Confirmed gap |
|---|---|---|
| Organization skill authoring | monocloud `machine/agents/skills/`; `machine/agents/render.py` and its manifest | Project copies are generated, but older global publication checkouts remain divergent and must not be overwritten. |
| Reviewed upstream intake | monocloud `machine/agents/intake.py`, `skills-upstream.lock.json`, ownership ledger | Implemented under review; empty ownership state does not claim any existing skill. |
| RPIV Praxis | `machine/rpiv/install.sh`, called by `machine/bootstrap.sh` | Installer follows the fork's `main`. Existing stale-Mac remediation is `9d91`; updating active workflow code is not authorized here. |
| RPIV Pi packages | `machine/agents/home/pi/settings.json`; home renderer | Declarations exist. Rendering configuration does not prove native packages were downloaded or loaded. |
| Impeccable | Installed `@fradser/pi-impeccable` 0.1.1 manifest and README | Native Pi extension is present on the inspected development machine but absent from the central package floor. Pre-1.0 compatibility needs explicit review. |
| Shared third-party portable skills | Dotfiles `.chezmoiexternal.toml` and link declarations | Existing upstream paths remain their owner; do not vendor another copy into this public library. |
| Cloud coding agents | `scripts/setup/cloud-agent.sh` and forwarding shims | Clones the old GitHub skills source; does not invoke the resource convergence needed to install the full documented stack. |
| Hermes third-party skills | `infra/hermes-vm/install-third-party-skills.sh` | Refresh replaces destination trees. Presence-only checks do not establish ownership, digest, provenance or preservation of edits. Do not extend this replacement behavior to research skills. |

The public README now separates the published snapshot from current internal operating policy. Historical Beads instructions are cataloged as historical, not selected for a current profile.

## Ownership contracts

### Organization-authored research skills

- Author and review in monocloud `machine/agents/skills/` unless a specific reviewed ownership handoff says otherwise.
- Keep its existing renderer as the only writer of generated project skill copies.
- Record an explicit research-profile selection, source revision and complete-tree digests for external consumers. Use the existing lock/ledger model rather than silently treating a filename as ownership.
- Include complete supporting trees and modes. Do not export a main Markdown file while dropping its references, scripts, attribution or license files.
- Preserve installed local edits and unknown ownership. Refuse a conflicting update and report what must be reconciled; do not force, reset, delete or fabricate a ledger to make it pass.
- Publishing selected instructions to this public repository is a distinct reviewed step. Installing private organizational instructions on an authorized Hermes host is not public-export approval.

The initial research profile selects the current canonical versions of `research-notes`, `research-reviewer`, `research-process-reset`, `evidence-crawl`, `exclusions-manager`, `writing-exclusions`, `investment-analysis`, `fact-checker`, `compliance-analyst`, `ecic-brand`, `ethical-editorial-gate`, `buzz-saw-editor`, `anti-racist-review`, `veil-of-ignorance` and `r2-lake-sink-guardrails`. These names are a proposed internal deployment selection, not a statement that every one is published in this repository.

`pipeline-runner` is a separate opt-in operating capability, not part of the default instruction-only research profile. Neither selection grants credentials or authorizes execution. Each consumer must still pass ownership and effective-name checks before installation.

Before releasing a profile, check required companion skills and host tools as part of its reviewed dependency closure. For example, `ethical-editorial-gate` requires `buzz-saw-editor` as well as the veil and anti-racist passes. Missing required companions must fail readiness; optional capabilities must be reported separately, not installed or authorized merely because a skill mentions them. Restrictive/internal licenses remain private unless an authorized publication review permits otherwise.

### Upstream packages and shared portable libraries

- RPIV skills/extensions and workflow execution remain upstream packages. Organizational workflow packs remain renderer-owned configuration, not another copy of upstream skill code.
- Add the inspected Impeccable Pi package to the existing central declaration using an explicit reviewed version. Keep its native tool and command interface; do not copy its procedure internals into skill directories.
- The native Pi adapter is not the Hermes or Claude Code adapter. Verify any non-Pi Impeccable artifact, host behavior and license before adding it through the existing portable-skill owner.
- Retain upstream notices and licenses. The public repository's CC BY-NC license does not relicense upstream MIT or Apache-2.0 material.
- Shared portable skills retain one canonical home tree. Do not add redundant Pi copies or alias a package-provided skill into a competing directory.

### Consumer context

One canonical authoring source can have generated or deployed copies, but there must be one selected resource per skill name in a host's effective loading context.

Do not solve missing global research skills by blindly adding a global Pi copy alongside its project copy. Verify global-plus-project-plus-package discovery and select the appropriate scope/adapter. Preserve intentional host differences and personal experimental additions.

## Setup behavior

### Local developer machines

Keep `machine/bootstrap.sh` and its documented plan as the entrypoint. Keep the home renderer responsible for owned configuration; add selected native resource installation/verification using the host's supported package manager.

Package installation must use the central declarations, not a separately maintained version list. It must preserve personal additions and unrelated settings. Do not implicitly update every experimental package or change providers, models, credentials or project trust while installing resources.

Keep dotfiles changes in its canonical source checkout and review them before targeted application. Do not apply broadly over known drift.

### Cloud coding agents

Use `scripts/setup/cloud-agent.sh`; keep Cursor/Claude/other platform shims as forwarders, not duplicate installers.

- Explicitly select applicable resources for the host. A Claude/Factory sandbox must not acquire Pi extensions merely because they appear in the catalog.
- Wire the selected RPIV Praxis/native-resource/configuration steps into the canonical execution list, not a README command that nothing invokes.
- Reuse canonical project research skills already present in the checkout. Do not create an old GitHub publication clone as a second authoring source.
- Distinguish configuring resources, installing packages and checking loaded resources. A required selected dependency that cannot be installed or verified is a nonzero result, not an unqualified `ready` message.
- Do not install private local-machine provider/MCP settings into a cloud host as a side effect of resource installation.
- Setup may perform deterministic checks. It must not launch LLM stages, alter account integrations or spend a workflow budget.

### Hermes

Keep Hermes provisioning separate from the cloud-sandbox bootstrap. Do not run a whole sandbox bootstrap against a long-lived Hermes host.

- Consume explicitly selected, compatible portable research skills from a clean, reviewed source checkout at a full pinned revision.
- Reuse ownership-aware staged installation and the existing private state/ledger model. Fetch/authentication belongs to provisioning; validation/intake remains independently testable without network access.
- Refuse unknown ownership, changed installed content, unsafe trees and duplicate effective skill names before writing.
- Preserve Hermes-authored learning, personal material and unrelated skills. Centrally owned research skills must not enter Hermes's authored/publication selection accidentally.
- Native RPIV Praxis CLI/MCP use requires its own runtime and SDK authentication. A portable skill install does not establish those prerequisites; Pi extensions do not run inside Hermes by copying their files.
- Verify inside the container's actual mounted discovery path. Presence is one check, not the full acceptance result.
- Do not activate export hooks, service units or public uploads as a side effect of installing a research profile.

### Product application runtimes — deferred

Document the eventual consumer boundary, but ship no product adapter in this phase. A later design must specify eligible skill profiles, authenticated access, revision pinning, data isolation, execution authority, updates and audit records. It must not treat instruction distribution as permission to run research or expose private material.

## Verification Notes

These requirements must route into implementation tests and real activation checks; a passing parser or helper alone is insufficient:

- **V1 — catalog integrity:** every published skill tree is classified; every relative link resolves; historical tracking instructions are excluded from current profiles.
- **V2 — declaration versus installation:** test selected package resolution and installed identity/version separately from a renderer's declaration check.
- **V3 — no execution side effects:** setup checks do not start RPIV stages, SDK sessions or research jobs. No provider/account changes, credential output or experimental-package updates.
- **V4 — cloud wiring:** the real cloud execution list invokes the selected installer/checker; setup changes affect the tests that protect them. Missing required tools fail visibly.
- **V5 — ownership refusal:** unknown destinations, changed owned bytes, source/remote changes and conflicting effective names are red. Unrelated skills and experiments remain intact.
- **V6 — source integrity:** pin and digest checks include support files, modes, links, bounds and scanning. An unavailable required scanner is a failure, not a clean scan.
- **V7 — failure and recovery:** exercise failed fetch/install/scan and precommit rollback paths; distinguish a committed installation with cleanup pending from a rolled-back failure.
- **V8 — effective loading:** probe applicable local, cloud and Hermes discovery paths with a harmless fixture. Verify version and duplicate-name diagnostics, not only `SKILL.md` presence.
- **V9 — privacy and authority:** no research data or personal learning is published; central research ownership and Hermes authorship do not overlap implicitly.
- **V10 — honest activation:** report source tests, installed checks, authenticated end-to-end delivery and actual host activation separately. Product-runtime integration stays deferred.

## Implementation slices and success criteria

### Slice 1 — public overview and ownership contract

Success criteria:

- README classifies all published skill trees and explains RPIV/Praxis, native Impeccable, portable libraries and tool dependencies.
- Relative-link and catalog-coverage checks pass; license description matches the repository license.
- Current gaps, historical material and unactivated behavior are labeled, not represented as deployed features. Covers V1 and the documentation portion of V10.

### Slice 2 — canonical native declarations and local/cloud setup

Success criteria:

- Impeccable is declared once in the existing native package floor with its reviewed version and pre-1.0 caveat.
- Existing setup entrypoints invoke selected native installation and deterministic checks; applicable required failures are nonzero.
- Tests demonstrate actual selection/identity checks, effective discovery, preserved settings/experiments and absence of workflow/provider/account side effects. Capture setup output with synthetic credential canaries and require that no canary is emitted.
- Setup-source changes trigger their tests in CI. Covers V2, V3, V4 and the local/cloud portion of V8.

### Slice 3 — centrally owned research profile and Hermes consumer

Success criteria:

- Research-profile source, revision, full-tree digests and ownership are explicit and version controlled.
- The default profile excludes historical tracking skills and the opt-in pipeline runner. Required companion-skill/tool checks fail when a mandatory dependency is missing; conditional capabilities remain explicitly unavailable rather than silently activated.
- Tests observe refusal for unknown ownership, local edits, wrong source/pin, unsafe trees, missing scanner and conflicting names; unrelated private learning survives.
- Failure/rollback/cleanup behavior is exercised; a harmless fixture is verified through the actual container discovery path before any live claim.
- No central research skill is silently selected for Hermes-authored publication. Covers V5, V6, V7, V9 and the Hermes portion of V8.

### Slice 4 — review and separately verified activation

Success criteria:

- Independent substantive findings are reproduced and resolved; applicable quality gates execute without bypasses.
- Runbooks identify exact installation/update/verification/rollback owners and keep separate evidence for source tests, installed checks, authenticated end-to-end delivery and actual host activation. A helper/fixture result cannot fill a live-delivery or activation field.
- Any live update is separately reviewed, avoids active RPIV runs and preserves dirty libraries. Coordinate `9d91` rather than duplicating its stale-install task.
- No product-runtime changes or implicit public publication occur. Covers V10 and live rechecks of V3, V8 and V9.

## Non-goals and limits

No all-skills auto-install, public export of private research content, reset of dirty libraries, universal cross-harness extension format, new sync service, new renderer, app-runtime adapter, or automatic workflow execution.

Ownership-aware exception rollback is not crash/power-loss journaling. Configuration checks do not prove package execution. Local fixtures do not prove authenticated remote delivery. A writable Forgejo repository does not automatically move GitHub/dotfiles consumers or install a reverse mirror.

## Docs-slice review record

Independent coverage review found three routing omissions: historical-skill exclusion, credential-output tests and separate authenticated-delivery/activation evidence. The Slice 2–4 success criteria now require these explicitly.

A bounded codebase-fit review found a missing required companion: `ethical-editorial-gate` requires `buzz-saw-editor`. The initial profile now includes it, with an explicit missing-dependency failure requirement. That review reached its turn limit; it is not represented as an exhaustive audit.

One-off executed documentation checks cover all 40 published skill trees, 43 relative links and the README navigation anchors. Negative controls reject the previous README, a removed research entry, a broken file link and a broken heading anchor. The proposed public documents passed a gitleaks scan. These are docs-slice checks, not a shipped CI guard or proof of native-package installation, authenticated delivery or live host activation.
