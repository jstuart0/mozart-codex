## Checklist

- [ ] I ran the edition parity checklist and it exited 0 (the commands below, from `CONTRIBUTING.md`).

A maintainer runs it from a full checkout of this repository, at its root; it cannot be run from an installed skill, because the install copies `.codex/agents` and `.codex/skills` only and the checker, `tests/parity/` and `tests/policy/` are not among them.

```sh
python3 scripts/check-edition-text.py selftest
python3 scripts/check-edition-text.py --edition codex --root . \
  --expect-rows 124 \
  --expect-source-rows 29 \
  --expect-ids 1a34b64c5f7018a18286e5b80def8afa7bebdf07e0ab4c61191a0b06a00df062 \
  --expect-table-sha256 851336ba715010ad90eae55c4679d883825a676e169d8bb4fad4696379688663 \
  --expect-policy-sha256 e43620dde3e5c381f703ed9f07a9eda9540108bdb931c1526e79e47f190c65eb \
  --expect-reader-sha256 76fc6bb130496bb2fc43b4d2eaddf35f75895825cec38d2b28da0e1a9eeb81f7
```
