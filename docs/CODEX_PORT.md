# Porting mozart to OpenAI Codex CLI

Status: design + proof-of-concept (jackson). Not yet a shipped port.
Verified against Codex CLI v0.142.0 (June 22, 2026) docs.

## Verdict

Portable, and far more cleanly than it would have been before Codex CLI grew
native subagents. As of v0.142.0 Codex has the three primitives mozart's
architecture rests on: custom subagent definitions, parallel fan-out, and
spawn-depth control. The orchestration *methodology* (pipeline stages,
TINY/STANDARD/HEAVY tiering, gates, state-file + flow-sketch artifacts) is
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
| `.claude/agents/*.md` (YAML front matter + markdown body) | `.codex/agents/*.toml` (`~/.codex/agents/` personal, `.codex/agents/` project) | Body ports into `developer_instructions = """..."""`. |
| `Task(subagent_type="harry")` | natural-language spawn; Codex resolves by the agent's `description` | Prompt-driven, not an explicit API call. |
| Parallel reviewer fan-out | `agents.max_threads` (default 6) | Compatible with mozart's ~3–4 concurrency cap. |
| Top-level-only; subagents can't spawn | `agents.max_depth` (default 1) | Exact match: mozart at depth 0 → specialists at depth 1; specialists don't spawn. |
| `SendMessage` (continue live agent) | orchestrator "routing follow-up instructions" to an existing thread | See "Open question, resolved" above. |
| `/mozart` slash command + skill | Codex **skill** (custom prompts are deprecated in favor of skills) | Skills support implicit + explicit invocation and ship in-repo. |
| `tools: Read, Grep, Glob, Edit, Write, Bash` | `sandbox_mode` (`read-only` vs `workspace-write`) + `mcp_servers` | Codex has no per-tool allowlist; capability is governed by sandbox mode. |
| `model: sonnet/opus` | `model` + `model_reasoning_effort` | See model map below. |
| `CLAUDE.md` (repo instructions) | `AGENTS.md` | Concatenated root→cwd, nearer overrides. |
| jcodemunch MCP (code-aware index) | `[mcp_servers.NAME]` in the agent TOML or `config.toml` | Same MCP server; declared per-agent or globally. |
| State file / flow sketch / tickets | unchanged | Plain files + shell; harness-agnostic. |

### Confirmed agent TOML schema

```toml
name = "..."                          # required — identifier
description = "..."                   # required — when Codex should use it
developer_instructions = """..."""    # the persona body
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
3. `ToolSearch` / "deferred tool" friction → "MCP/skill load" framing.
4. "single parallel tool-call message" → "parallel fan-out (`max_threads`)".
5. Tool-list front matter → `sandbox_mode` (read-only for reviewers/auditors;
   workspace-write for jackson, harry, ruby, scott).
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

## Work breakdown

1. **POC (this doc):** translate jackson → `.codex/agents/jackson.toml`. ✅
2. Translate the other 13 personas (mechanical, per the rules above).
3. Author the mozart conductor as `.codex/agents/mozart.toml` +
   the entry-point skill.
4. Swap the cross-model reviewer in mozart's body: `codex exec` → `claude -p`.
5. Add `config.toml` `[agents]` defaults (max_threads, max_depth) and document
   `~/.codex` vs project install in INTEGRATION.md.
6. Live-validate the four risks above on a TINY DELIVER run.

POC artifact: `codex/agents/jackson.toml`.
