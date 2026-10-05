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

## Release checklist: edition parity

This port has no CI, so edition parity is checked by a person before a release.
`scripts/check-edition-text.py` and `tests/parity/editions.tsv` are copies of the
mozart-orchestration reader and table, and `tests/policy/` holds the text they pin.
Run, from the repository root:

```sh
python3 scripts/check-edition-text.py selftest
python3 scripts/check-edition-text.py --edition codex --root . \
  --expect-rows 100 --expect-source-rows 29 \
  --expect-ids 70350680cfa8f5496e57181b41699f65357c969a846cebf06232084101132547 \
  --expect-table-sha256 61fd511e07ad783c066a8c531fff84ce47263941223f71977108afd9242f49df \
  --expect-policy-sha256 41a2eb5e5b4f8aa7eb94d519e0885c801c49af363153a5ef874fc1f2ba7ed973 \
  --expect-reader-sha256 76fc6bb130496bb2fc43b4d2eaddf35f75895825cec38d2b28da0e1a9eeb81f7
```

Both commands must pass. The checklist passes only after the layout rows (the
inline state, ledger and conductor skeletons, and the lens date) have landed; until
then the second command fails on those rows.
