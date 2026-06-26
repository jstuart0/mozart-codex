# Changelog

All notable changes to mozart-codex are documented here.

## [Unreleased]

### Added
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
