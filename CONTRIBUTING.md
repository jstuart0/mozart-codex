# Contributing to mozart-codex

Thanks for your interest. This repo is the OpenAI Codex CLI port of the mozart
orchestration system.

## Ground rules

- **Personas are faithful ports.** The specialist personas in `.codex/agents/`
  and the conductor in `.codex/skills/mozart/` are translated from the Claude
  Code edition. When changing behavior, keep the two editions semantically in
  sync unless the change is harness-specific (spawning, continuation, the
  cross-model reviewer). Document harness-specific divergences in
  `docs/CODEX_PORT.md`.
- **Don't commit working artifacts.** `thoughts/` (mozart's state files, flow
  sketches, plans, research briefs) and your live `config.toml` are gitignored.
  Keep them out of commits.
- **TOML hygiene.** Agent definitions use TOML literal multiline strings
  (`'''…'''`) for `developer_instructions` so backslashes and quotes in the
  prose survive. Validate any edited agent file parses:
  `python3 -c "import tomllib,sys; tomllib.load(open(sys.argv[1],'rb'))" .codex/agents/<name>.toml`

## What to work on

- Validating the open questions in `docs/CODEX_PORT.md` against live Codex runs
  (spawn determinism, thread-continuation granularity, `developer_instructions`
  size limits, skill-as-entry-point).
- Keeping the model map current as Codex models evolve.
- Bug reports from real orchestration runs (attach the flow sketch if you can).

## PRs

Small, focused PRs. Describe what changed and why; if it touches a persona, note
whether the Claude edition needs the mirror change. No AI-attribution noise in
commit messages.
