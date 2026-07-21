# mozart-codex

A multi-agent software-delivery orchestration suite for **OpenAI Codex CLI**.

`mozart` is a senior delivery *conductor*: it doesn't write the code itself, it
decides which specialist runs, when, and in what order — then spawns them as
Codex subagents and drives the work end-to-end. This repo is the Codex CLI port
of the mozart orchestration system (originally built as a Claude Code plugin).

> Status: early port. The personas and pipeline are complete; the harness
> mapping (spawning, continuation, the cross-model reviewer) is documented in
> [`docs/CODEX_PORT.md`](docs/CODEX_PORT.md) and still being validated on live
> Codex runs. Issues and PRs welcome.

## What it does

Mozart orchestrates work across five shapes:

- **DELIVER** — build a feature: research → plan → review → implement → validate → ship → document
- **AUDIT** — review against a goal: discover → fan-out → synthesize → optionally remediate
- **DIAGNOSE** — investigate a failure: intake → investigate → present findings → optionally remediate → optionally publish post-mortem
- **OPERATE** — change or debug a live system: intake+context pin → recon → change plan → pre-flight (dry-run+snapshot) → apply → verify observed → record rollback. Installs, config changes, and infra mutations applied straight to the running cluster/host rather than through a git pipeline; verified empirically, reversed by a recorded rollback (not `git revert`)
- **EVAL** — evaluate mozart's own field performance from past campaign artifacts and improve the configuration (see the `mozart-eval` skill and `docs/EVAL.md`)

At intake it **tiers** the task (TINY / STANDARD / HEAVY) to right-size the gates,
classifies the project (GREENFIELD / BROWNFIELD), and recognizes when a request
is genuinely a single agent's job — routing it directly instead of imposing the
full pipeline.

## The orchestra

| Agent | Role |
|---|---|
| **mozart** | Conductor — runs the pipeline, spawns the rest (this is the skill you launch) |
| sarah | Technical research / prior-art |
| harry | Planning architect |
| bob | Plan review (architecture) |
| dexter | Code-health audit |
| xander | Security audit |
| otto | Infra / Kubernetes review (+ OPERATE change-plan author) |
| hank | Ops executor — applies changes to live infrastructure (OPERATE) |
| ruby | UI/UX design + frontend |
| ian | Change-impact / blast-radius |
| librarian | "Does this already exist?" prior-code check |
| tessa | Test strategy / TDD contract |
| dick | Bug / regression investigation |
| jackson | Implementation (builds + fixes) |
| valerie | Verification against the plan |
| scott | Documentation |
| codebase-locator / -analyzer / -pattern-finder, web-search-researcher | Fast retrieval helpers |

Each specialist is a Codex subagent defined in [`.codex/agents/`](.codex/agents/).
Mozart itself is a skill in [`.codex/skills/mozart/`](.codex/skills/mozart/).

## How it maps to Codex CLI

Mozart's architecture rests on three Codex primitives (v0.142.0+):

- **Subagents** — `.codex/agents/*.toml` definitions, spawned by description match.
- **Parallel fan-out** — bounded by `agents.max_threads` (default 6).
- **Spawn depth** — `agents.max_depth = 1`: the conductor spawns specialists; specialists don't sub-spawn. This is mozart's exact contract.

The **independent cross-model reviewer** inverts from the Claude edition: where
that version shells out to `codex exec`, this one uses the **`claude` CLI**
(`claude -p`) as the second-model auditor at the plan and diff gates. Same value
(a different model family auditing the work), opposite binary.

Full primitive mapping, the model map, and known open questions:
[`docs/CODEX_PORT.md`](docs/CODEX_PORT.md).

## Install

```bash
git clone https://github.com/<you>/mozart-codex.git
cd mozart-codex

# Project-scoped: Codex picks up ./.codex/agents and ./.codex/skills automatically
# when you run it from a repo that vendors these. Or install globally:
cp -r .codex/agents/*    ~/.codex/agents/
cp -r .codex/skills/*    ~/.codex/skills/

# Merge the orchestration defaults into your Codex config:
cat config.toml.example   # then copy the [agents] block into ~/.codex/config.toml
```

Requirements:
- Codex CLI v0.142.0+ (native subagents, `[agents]` config).
- The `claude` CLI on `PATH` and authenticated, if you want the cross-model
  review gates (optional — mozart degrades gracefully without it).

## Use

Launch mozart at the top level of a Codex session and hand it the task:

```
mozart: add SSO via our IdP to the admin panel
mozart: audit this repo for tech debt
mozart: investigate why staging queries are slow
mozart: resume the campaign at thoughts/shared/plans/<slug>.state.md
```

Mozart runs intake first (shape, tier, mode, slug), creates a state file and a
flow sketch, then conducts the pipeline — narrating each agent invocation so you
can follow along.

## Configuring your repo

Mozart adapts to your **ticketing**, **documentation**, and **code-retrieval**
setup via stanzas in the consuming repo's `AGENTS.md`. See
[`INTEGRATION.md`](INTEGRATION.md) for the contract and per-system templates. If
a stanza is absent, the corresponding behavior is skipped or falls back to a
sensible default.

## Repo layout

```
.codex/
  agents/*.toml            specialist subagent definitions
  skills/mozart/SKILL.md   the conductor entry point
config.toml.example        [agents] defaults + reviewer notes
INTEGRATION.md             ticketing / docs / code-retrieval contract
PIPELINE.md                full stage-by-stage pipeline reference
LEARNINGS.md               append-only cross-project field-notes protocol
docs/CODEX_PORT.md         Claude→Codex port rationale + mapping
```

## Relationship to the Claude Code edition

This is a faithful port. The orchestration *methodology* — pipeline stages,
tiering, gates, state-file and flow-sketch artifacts, ticket lifecycle,
narration cadence — is harness-neutral and identical to the Claude edition. Only
the spawn / continue / reviewer plumbing differs, per `docs/CODEX_PORT.md`.

## License

MIT — see [LICENSE](LICENSE).
