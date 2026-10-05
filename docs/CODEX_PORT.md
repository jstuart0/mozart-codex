# Porting mozart to OpenAI Codex CLI

Status: design + proof-of-concept (jackson). Not yet a shipped port.
Verified against Codex CLI v0.142.0 (June 22, 2026) docs.

## Verdict

Portable, and far more cleanly than it would have been before Codex CLI grew
native subagents. As of v0.142.0 Codex has the three primitives mozart's
architecture rests on: custom subagent definitions, parallel fan-out, and
spawn-depth control. The orchestration *methodology* (pipeline stages,
TINY/LIGHT/STANDARD/HEAVY tiering, gates, state-file + flow-sketch artifacts) is
harness-agnostic and ports unchanged. Only the spawn / continue / entry-point
plumbing changes.

## Open question, resolved: programmatic continuation

The `SendMessage`-style "continue a live agent with context intact" capability
(the iteration fix on the Claude side) has a Codex analog, but it is expressed
at the orchestration layer rather than as a tool the persona calls.

Codex docs: *"Codex handles orchestration across agents, including spawning new
subagents, **routing follow-up instructions**, waiting for results, and closing
agent threads."* The `/agent` command is for the **human** to inspect/steer/stop
threads; it is not the orchestrator's mechanism. The orchestrator pattern is
turn-based: subagents run, Codex returns a consolidated result, the parent
issues follow-up instructions, and Codex routes them (continuing the existing
thread). 

Implication for mozart's iteration loops (harry revise, jackson reconcile,
valerie incremental, LOOP-IN feedback): the *goal* — preserve the agent's loaded
context across a revision round — is achievable, because Codex routes follow-ups
to the existing thread. But mozart-on-Codex expresses it as a natural-language
follow-up ("have harry revise the plan with these findings") rather than an
explicit `SendMessage(harry, ...)` call. Less deterministic; same outcome.

## Primitive mapping

| Mozart (Claude Code) | Codex CLI | Notes |
|---|---|---|
| `.claude/agents/*.md` (YAML front matter + markdown body) | `.codex/agents/*.toml` (`~/.codex/agents/` personal, `.codex/agents/` project) | Body ports into `developer_instructions = '''...'''`. |
| `Task(subagent_type="harry")` | natural-language spawn; Codex resolves by the agent's `description` | Prompt-driven, not an explicit API call. |
| Parallel reviewer fan-out | `agents.max_threads` (default 6) | Compatible with mozart's ~3–4 concurrency cap. |
| Top-level-only; subagents can't spawn | `agents.max_depth` (default 1) | Exact match: mozart at depth 0 → specialists at depth 1; specialists don't spawn. |
| `SendMessage` (continue live agent) | orchestrator "routing follow-up instructions" to an existing thread | See "Open question, resolved" above. |
| `/mozart` slash command + skill | Codex **skill** (custom prompts are deprecated in favor of skills) | Skills support implicit + explicit invocation and ship in-repo. |
| `tools: Read, Grep, Glob, Edit, Write, Bash` | `sandbox_mode` (`read-only` vs `workspace-write`) + `mcp_servers` | Codex has no per-tool allowlist; capability is governed by sandbox mode. Verified empirically, not just asserted: a depth-0 `read-only` session's `apply_patch` was refused with *"patch rejected: writing is blocked by read-only sandbox; rejected by user approval settings"* — see the capability-vs-claim parity campaign (`.mozart/plans/active/2026-09-12-deliver-capability-claim-parity.md`, Phase 0a). That denial is conjunctive and was observed under `approval: never`, so the confirmed claim is narrower than "read-only always denies." |
| `model: sonnet/opus` | `model` + `model_reasoning_effort` | See model map below. |
| `CLAUDE.md` (repo instructions) | `AGENTS.md` | Concatenated root→cwd, nearer overrides. |
| jcodemunch MCP (code-aware index) | `[mcp_servers.NAME]` in the agent TOML or `config.toml` | Same MCP server; declared per-agent or globally. |
| State file / flow sketch / tickets | unchanged | Plain files + shell; harness-agnostic. |

### Confirmed agent TOML schema

```toml
name = "..."                          # required — identifier
description = "..."                   # required — when Codex should use it
developer_instructions = '''...'''    # the persona body
model = "gpt-5.3-codex"               # optional — inherits session if omitted
model_reasoning_effort = "high"       # optional — low | medium | high
sandbox_mode = "workspace-write"      # read-only | workspace-write
nickname_candidates = ["Atlas"]       # optional — display names

[mcp_servers.someServer]              # optional — per-agent MCP
url = "https://..."

[[skills.config]]                     # optional — per-agent skills
path = "/path/to/SKILL.md"
enabled = false
```

`config.toml` `[agents]` keys: `max_threads` (6), `max_depth` (1),
`job_max_runtime_seconds`.

### Model map (Claude → Codex)

| Mozart role | Claude model | Codex model | Effort |
|---|---|---|---|
| Builders (jackson) | sonnet | `gpt-5.3-codex` | high |
| Conductor (mozart) | opus | `gpt-5.4` | high |
| Deep reviewers (bob, dexter, xander, harry) | sonnet/opus | `gpt-5.4` | high |
| Fast scan/synthesis (locators, finders) | sonnet | `gpt-5.3-codex-spark` | medium |

## Cross-model auditor inversion

Mozart-on-Claude uses `codex exec` as the **independent cross-model reviewer**
(DELIVER stages 5 and 10 — "codex r1/r2"). On Codex this inverts: **Claude
becomes the external auditor**, invoked via the `claude` CLI for the same
cross-model review. The "External tool execution discipline" section (background
invocation, ~5-min polling, 30-min hard cap, escalation path) ports verbatim —
just point it at `claude -p` instead of `codex exec`. The value (a second model
family auditing the first's work) is preserved; only the binary changes.

## Per-agent translation rules

The persona *body* ports nearly verbatim. Mechanical swaps applied per agent:

1. `CLAUDE.md` → `AGENTS.md`.
2. Tool nouns `Read`/`Grep`/`Glob`/`Edit`/`Write` → generic "file read / search
   / edit" (Codex's built-ins); keep the discipline, drop the Claude tool names.
   Swept to completion by the capability-vs-claim parity campaign
   (`.mozart/plans/active/2026-09-12-deliver-capability-claim-parity.md`): the
   last 6 stale bold tool-noun references (`**Bash**`, `**WebFetch**`) on
   `dick.toml` and `tessa.toml` are gone; 0 remain across the 20 personas that
   existed at the time of that sweep. `nina.toml` (added later) carries none
   either. NOTE the sweep targeted the **bold** form: backticked `Read` still
   appears in 12 personas, which is residue from that sweep rather than a new
   divergence, and nina inherits the same convention.
3. `ToolSearch` / "deferred tool" friction → "MCP/skill load" framing.
4. "single parallel tool-call message" → "parallel fan-out (`max_threads`)".
5. Tool-list front matter → `sandbox_mode` (read-only for reviewers/auditors;
   workspace-write for bob, hank, harry, jackson, percy, ruby, scott, tessa —
   8 of 21 personas, not 4).
6. "bundled PIPELINE.md / LEARNINGS.md / mozart persona" → same files shipped
   under `.codex/` alongside the agents.
7. `codex exec` review references (in mozart.md) → `claude -p` review.

Everything else — the disciplines, contract checks, gates, narration cadence —
is harness-neutral and stays as written.

## Risks / things to validate live

- **Spawn determinism.** Codex spawning is prompt-driven; mozart's precise
  "spawn exactly bob + librarian + xander in parallel" depends on the model
  reliably translating intent into spawns. Looser than `subagent_type`.
- **Continuation granularity.** Confirm in practice that follow-up routing
  actually continues the *same* thread (context intact) vs silently re-spawning.
  If it re-spawns, the iteration loops degrade to brief-from-artifacts (the
  pre-fix behavior) — acceptable but worth knowing.
- **`developer_instructions` size.** mozart.md is ~143KB. Confirm no field-size
  limit truncates it; if so, split into `developer_instructions` + a bundled
  skill/AGENTS.md the conductor reads.
- **Skill as entry point.** Verify a Codex skill can act as the `/mozart`
  top-level entry that then orchestrates, mirroring the slash-command role.
- **Missing upstream commit `e9232c0` (disclosed, not yet ported).** This port
  predates orchestration's `.mozart/` artifact-root convention, its worktree
  isolation, and hank's version-resolution step — and correspondingly has no
  stage 12b or its lint check (`missing-12b`). `scripts/mozart-lint.sh` and
  `scripts/mozart-metrics.sh` already read both `.mozart/` and
  `thoughts/shared/` so a target repo on either convention lints cleanly; the
  rest (worktree isolation, the version-resolution gate, 12b itself) is
  scoped as its own follow-up campaign.
- **Lint deviations from upstream (disclosed, three).** `scripts/mozart-lint.sh`
  differs from upstream's code in exactly three places: no Check I
  (`missing-12b`), `Claude|Codex` matched and reported as `review-drift`, and the
  nothing-to-lint message wording. The default of `MOZART_LINT_LENS_SINCE` is
  upstream's, `2026-10-04`: this port's manual now tells its conductor to record a
  HEAVY surface and both lenses on every phase row, so the dated lens-record rules
  apply to campaigns slugged on or after that day.

## Work breakdown

1. **POC (this doc):** translate jackson → `.codex/agents/jackson.toml`. ✅
2. Translate the other 13 personas (mechanical, per the rules above). ✅ — shipped
   as 21 personas at `.codex/agents/*.toml`.
3. ~~Author the mozart conductor as `.codex/agents/mozart.toml`~~ — **stale.**
   There is no `mozart.toml` and there will not be one: the conductor shipped as
   a skill, `.codex/skills/mozart/SKILL.md`, not an agent. ✅ — done, differently
   than planned here.
4. Swap the cross-model reviewer in mozart's body: `codex exec` → `claude -p`.
5. Add `config.toml` `[agents]` defaults (max_threads, max_depth) and document
   `~/.codex` vs project install in INTEGRATION.md.
6. Live-validate the four risks above on a TINY DELIVER run.

POC artifact: `codex/agents/jackson.toml`.

## Skeletons stay in the skill

mozart-orchestration moved its state, ledger, conductor, flow and report skeletons out of the manual into `TEMPLATE-*.md` files, so the conductor reads each one only when it writes one. This port installs a single skill file, so the skeletons stay inline in `.codex/skills/mozart/SKILL.md`: three fenced blocks under *State file format* (state, ledger, conductor), one under *Pipeline flow sketch* and one under stage 13. The ledger and conductor blocks are byte-identical to the source's template files, and the parity table checks that; the state block differs from the source's only by this port's artifact paths, reviewer names and stage list (no stage 12b). The read saving the split buys in the source is not delivered here.

## Release checklist

This port has no CI, so edition parity is checked by a person before a release. `scripts/check-edition-text.py` and `tests/parity/editions.tsv` are copies of the mozart-orchestration reader and table, and `tests/policy/` holds the text they pin.

A maintainer runs it from a full checkout of this repository, at its root; it cannot be run from an installed skill, because the install copies `.codex/agents` and `.codex/skills` only and the checker, `tests/parity/` and `tests/policy/` are not among them.

```sh
python3 scripts/check-edition-text.py selftest
python3 scripts/check-edition-text.py --edition codex --root . \
  --expect-rows 105 \
  --expect-source-rows 29 \
  --expect-ids 8e17e6c20a48ea463eba20d0bb513a24f56e6f7c312c019b86de1da1305cd6a4 \
  --expect-table-sha256 bb82ff23269667613814859557cf4a712ea67854ee70e756d0c044d511b16a18 \
  --expect-policy-sha256 d84f21f4fade1d144cc82ab4a3bb1645edb22a31a9ca42c2751912a229f06e0f \
  --expect-reader-sha256 76fc6bb130496bb2fc43b4d2eaddf35f75895825cec38d2b28da0e1a9eeb81f7
```

Both commands must pass. The checklist passes only once every layout row has landed. After the table or the reader is re-copied from mozart-orchestration, run `python3 scripts/check-edition-text.py hashes --edition codex` and replace these six literals with its output, here and in `docs/CODEX_PORT.md` and `.github/pull_request_template.md`.
