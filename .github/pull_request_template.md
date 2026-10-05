## Checklist

- [ ] I ran the edition parity checklist and it exited 0 (the commands below, from `CONTRIBUTING.md`).

A maintainer runs it from a full checkout of this repository, at its root; it cannot be run from an installed skill, because the install copies `.codex/agents` and `.codex/skills` only and the checker, `tests/parity/` and `tests/policy/` are not among them.

```sh
python3 scripts/check-edition-text.py selftest
python3 scripts/check-edition-text.py --edition codex --root . \
  --expect-rows 100 \
  --expect-source-rows 29 \
  --expect-ids 70350680cfa8f5496e57181b41699f65357c969a846cebf06232084101132547 \
  --expect-table-sha256 6471999eadcbc89f18a55133b0bc6472e017437363d62c03e2e94414089861ae \
  --expect-policy-sha256 41a2eb5e5b4f8aa7eb94d519e0885c801c49af363153a5ef874fc1f2ba7ed973 \
  --expect-reader-sha256 76fc6bb130496bb2fc43b4d2eaddf35f75895825cec38d2b28da0e1a9eeb81f7
```
