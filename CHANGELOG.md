# Changelog

All notable changes to mozart-codex are documented here.

## [Unreleased]

### Added (parity sync with mozart-orchestration through 2026-07-09, `8972513`)
- **EVAL** — fourth work shape: mozart evaluates its own field performance from
  campaign artifacts. New `mozart-eval` skill, `docs/EVAL.md` (ledger schema +
  report format), EVAL pipeline section in the conductor.
- `scripts/mozart-lint.sh` — mechanical hygiene linter for campaign artifacts
  (status-vs-location drift, review-drift, duplicate stages, stale actives,
  stranded artifacts). Matches both `Claude` and `Codex` external-review naming.
- **Auto-TDD detection at intake** — mozart auto-sets the TDD flag when the
  scope hits money/authz/compliance correctness, crisp pre-specifiable
  contracts, concurrency/idempotency invariants, or a reproducible bug fix;
  disclosed on the intake flags line, explicit user decline always wins.
- **Ruby design bar** — checklist compliance is not design approval; admin and
  operator screens get the same bar; render-before-verdict rule with
  `STRUCTURAL-ONLY` labeling; greenfield plans need a design foundation before
  feature-UI phases. Mozart's stage-4 and mid-build ruby triggers widened to
  operator-facing screens and made additive to the dominant-risk lens. Ruby
  bumped `gpt-5.3-codex` → `gpt-5.4` (the pipeline's only design-taste gate).
- **Hang-proof external review** — closed-stdin + OS-enforced kill-timer
  (GNU `timeout` / perl `alarm`) on every `claude -p` invocation; prompt-echo
  and timer-death recognized as tool failures; CPU-time-based liveness checks.
- **Atomic campaign closeout** — one transaction: reconcile state in place,
  finalize flow sketch, glob-move all slug artifacts, close parent campaigns,
  propagate to the canonical checkout; lint as the final closeout act.
- **Cross-checkout resume safety** — freshness check across worktrees before
  trusting a local state file; `Authoritative checkout` and `Worktree` fields
  in the state template.
- **Jackson workspace identity preflight** — worktree/branch/interpreter
  verified before first edit, any commit, and any test run.
- **Tessa binding rule** — one real-dependency test per integration seam, or a
  named waiver; mocked-only coverage at a seam is a High finding.
- **Valerie hardening** — mechanism drift in scope; SIGNOFF must state the
  disposition of every open external-review Critical/High.
- Subagent context budget discipline for large-AGENTS.md repos (campaign
  context digest above ~1,000 lines).
- Iteration/reconciliation counters incremented at round launch; RE-AUDIT
  delta-audit mode; stage-list skip lines mandatory; edit-in-place state files.

### Added (initial port)
- Initial Codex CLI port of the mozart orchestration system.
- All 15 specialist personas as `.codex/agents/*.toml` subagent definitions
  (sarah, harry, bob, dexter, xander, otto, ruby, ian, librarian, tessa, dick,
  jackson, valerie, scott) plus the retrieval helpers (codebase-locator,
  codebase-analyzer, codebase-pattern-finder, web-search-researcher).
- `mozart` conductor as a Codex skill (`.codex/skills/mozart/SKILL.md`).
- Cross-model reviewer inversion: `claude -p` as the independent second-model
  auditor at the plan and diff gates (replacing `codex exec` from the Claude
  edition).
- `config.toml.example` with `[agents]` orchestration defaults.
- `INTEGRATION.md` (ticketing / documentation / code-retrieval contract, keyed
  to `AGENTS.md`), `PIPELINE.md`, `LEARNINGS.md`.
- `docs/CODEX_PORT.md` — port rationale, Claude→Codex primitive mapping, model
  map, and open questions to validate on live runs.
