## Checklist

- [ ] I ran the edition parity checklist and it exited 0 (the commands below, from `CONTRIBUTING.md`).

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
