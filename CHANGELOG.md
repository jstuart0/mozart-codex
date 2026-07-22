# Changelog

All notable changes to mozart-codex are documented here.

## [Unreleased]

### Added (parity sync with mozart-orchestration, 2026-07-22: INCIDENT shape)
- **INCIDENT** — sixth work shape: respond to a **live outage** (service down or
  badly degraded *right now*). The time-critical form of DIAGNOSE — it
  **inverts** DIAGNOSE's "don't fix in the same pass" rule: mitigate first to
  restore service, race hypotheses in parallel, then durable-fix. Zero new
  agents — mozart is the incident commander (IC); responders (dick, hank, otto,
  xander, percy, scott) are all reused. New `## INCIDENT pipeline` section in the
  conductor: 7 stages (declare+triage → stabilize ‖ race hypotheses → converge →
  durable fix → verify recovery → blameless post-mortem), **SEV1/2/3** tiers (the
  INCIDENT tier axis, replacing TINY/STANDARD/HEAVY), and the parallelism
  discipline — read-only investigation fans out into hypothesis lanes, live
  mutation serializes through the one hand (hank).
- **Speed vs. rigor is *sequenced*, not chosen** — the reason it's a distinct
  shape. Mitigation runs gates-relaxed and logged `accepted-risk (incident)`
  with a rollback command; the durable fix runs full gates (DELIVER/OPERATE,
  repro-test-first, claude/ian/xander) once service is back.
- New state-file blocks: the `## Timeline (INCIDENT only)` append-only spine (the
  incident source of truth — survives crashes like the change ledger), the
  change-ledger heading widened to `## Change ledger (OPERATE + INCIDENT
  mitigations)`, the `INCIDENT-FULL` / `MITIGATE-ONLY` `Flow` values, and the
  `MITIGATE-ONLY` partial flow ("just get it back up" — stops after stage 3 + 5,
  durable fix deferred). Observability gate at declare-time: if the repo's
  `AGENTS.md` documents no monitoring/SLO stack, mozart surfaces that recovery
  can't be measured objectively and recommends an observability follow-up.
- **Persona touch-ups**: dick gains an INCIDENT hypothesis-lane mode (time-boxed,
  single parallel lane, report-to-timeline, don't block restore on perfect root
  cause); hank gains the *one* sanctioned exception to "never mutate without a
  snapshot" (restore-service can outrank snapshot under a declared incident — but
  rollback-command, context-verify, serialize, and verify-before-stacking still
  hold); scott owns the blameless post-mortem (action items, not attribution).
  Wired into PIPELINE.md (INCIDENT pipeline section, passthrough +INCIDENT,
  partial-flow +MITIGATE-ONLY, incident-timeline output path) and README.md (six
  shapes + INCIDENT). Agent count stays 16 (no new agent).

### Added (parity sync with mozart-orchestration, 2026-07-21: OPERATE shape + hank)
- **OPERATE** — fifth work shape: change or debug a **live system** directly
  (installs, config changes, infra mutations, hands-on debugging of running
  k8s / hosts / storage / DBs). The artifact is a state change to running
  infrastructure, not a git diff; verification is empirical (curl, logs,
  `get`), not CI; rollback is a recorded command against a snapshot, not
  `git revert` — which is why it's a distinct shape, not a DELIVER tier. New
  OPERATE pipeline section in the conductor (intake+pin → recon → change plan
  → pre-flight → apply → verify → record), OPERATE tiers/modes, the
  DELIVER-vs-OPERATE boundary test, DIAGNOSE/AUDIT → OPERATE routing, the
  `OPERATE-PLAN-ONLY` partial flow, the `OPERATE-FULL`/`OPERATE-PLAN-ONLY`
  state-file `Flow` values, and the `## Change ledger (OPERATE only)` state
  block (crash-safety spine: snapshot path + rollback command recorded before
  the apply). Wired into PIPELINE.md and README.md.
- **hank** (`.codex/agents/hank.toml`, gpt-5.4, workspace-write) — senior
  operations engineer, the hands-on counterpart to otto: the only agent that
  mutates live state. Executes changes against live infrastructure under a
  fixed loop — verify context → server-side dry-run → snapshot → apply one
  step at a time → verify observed (never expected) → record rollback. Runs
  OPERATE stages 4–6 and as a passthrough for one-off "just apply this" /
  "install X" / "restart the pod" requests.
- **otto promoted to OPERATE change-plan author** — in DELIVER he reviews
  infra-as-code; in OPERATE (stages 2–3) he authors the change plan (exact
  commands, per-step dry-run, snapshot step, rollback procedure, blast
  radius), and on HEAVY verifies the server-side dry-run + immutable fields at
  the pre-flight gate. hank executes what otto plans; he never designs the
  change himself. Passthrough routing gains hank rows; the conducted roster
  gains hank.

### Added (parity sync with mozart-orchestration, 2026-07-18: percy + findings ledger)
- **percy** (`.codex/agents/percy.toml`, gpt-5.4, workspace-write) — senior
  performance engineer, measurement-first: every finding carries a measurement
  he took (EXPLAIN, endpoint timing, bundle-size diff, profile) or a cited
  complexity argument tied to a named hot path; "could be slow" is not a
  finding, premature optimization is an explicit anti-pattern. Reviews the
  plan's performance *budgets* at stage 4 (p95 / query count / payload /
  bundle size on hot paths); measures slices at stage 8; leads
  performance/scaling AUDITs (bob + dexter demoted to support). Conditional
  triggers wired in the conductor and PIPELINE.md; k8s sizing stays with otto.
- **Findings ledger + escapes** (conductor state-file format) — one row per
  dispositioned Critical/High/Medium finding (stage, lens, severity,
  fixed/rejected/accepted-risk); rejected rows are kept as false-positive
  data. `## Escapes` block records `Traces-to:` links when a later DIAGNOSE
  or audit finds a defect the campaign shipped; dick's investigation template
  records the same link from the discovering side.
- **`scripts/mozart-metrics.sh`** — aggregates ledgers + escapes across all
  state-file layouts into the pipeline-economics table: catches by
  stage/lens/severity, false-positive rate per lens, escapes,
  defect-removal efficiency, catches-per-campaign by tier. Wired into EVAL
  stage 2; `docs/EVAL.md` gains the Pipeline economics section with
  gate-tuning decision rules and lower-bound / anti-Goodhart caveats.
- **Drift fixes**: PIPELINE.md roster carried the source repo's opus/sonnet
  model labels (now matches the actual TOML models) and predated tessa
  (rows added to roster + trigger tables).

### Added (parity sync with mozart-orchestration, 2026-07-16 coding-practice gap closures)
- **Toolchain baseline check** — new intake pre-flight gate 3: GREENFIELD
  campaigns (or any repo missing lint/format/type-check/test/CI) must open with
  a toolchain-bootstrap phase before feature phases; BROWNFIELD gets a surfaced
  triage decision. Per-phase gate fails when a diff touches a language with no
  configured mechanical check on GREENFIELD. Harry gains the matching
  toolchain-before-features sequencing rule + self-review item.
- **Mechanical secret scan at the per-phase gate** — gitleaks/trufflehog when
  installed, high-signal grep fallback otherwise; any hit is a gate failure,
  never "commit now, scrub later." Reviewer eyeballs demoted to backstop.
- **Xander: dependency vetting checklist** (provenance/typosquat, maintenance,
  advisories on the resolved version, license, transitive footprint + install
  scripts, pinning + lockfile-in-same-diff; scaled by change size) and a
  **CI/CD pipelines** checklist item (unpinned actions, over-broad workflow
  token permissions, `${{ }}` injection, `pull_request_target` + PR-head
  checkout, fork-PR secrets, cache poisoning). Manifest/lockfile and workflow
  diffs added as stage-4 and stage-8 xander triggers in the conductor and
  PIPELINE.md.
- **Jackson: Observability and Error handling sections** — structured logs with
  level discipline, correlation-ID propagation, no secrets/PII in logs,
  health+metrics on new long-running services; one error idiom per codebase,
  wrap-with-context, structured domain errors vs loud unexpected failures,
  log-once-where-handled, user-facing messages say what to do next.
- **Otto: observability wiring** in operational hygiene — new services ship
  with the documented monitoring stack and at least one down/crash-loop alert.

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
