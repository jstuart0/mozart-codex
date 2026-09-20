# Changelog

All notable changes to mozart-codex are documented here.

## [Unreleased]

### Added — nina, a cloud specialist, and a widened HEAVY trigger

Ported from `mozart-orchestration` as **`.codex/agents/nina.toml`**; the roster goes
**20 → 21** personas. **nina** reviews *assertions about how a cloud
provider behaves* — support or deprecation status, a quota or limit, a blocked or
permitted action, "cannot be moved", a permission conclusion — by resolving each
against a current provider source instead of recalling it. Her evidence base is AWS;
on Azure and GCP she applies the same method with no accumulated trap knowledge, and
live provider reads are AWS-only. She reviews and never mutates, never authors an
OPERATE change plan, and never issues a security severity.

Her read rules are a conjunction — an allowed verb, a non-denied bucket **and** a
projection matching a grammar allowlist with a declared value kind from a closed set
of seven — and they are carried from the upstream parity snippets **S22a** and **S22b**
**byte-exact, with zero divergence**. The rules are frozen as two contiguous spans with
a per-edition adjunct between them, so no frozen line names a tool this edition does not
have; the adjunct is the only place a tool name appears.

**`sandbox_mode = "read-only"` bars no provider API call.** It is a filesystem
setting; Codex has no per-tool allowlist. `nina.toml` says so in its own body, because
this is the edition most likely to be mistaken for enforced.

The review-role IAM skeleton that is the enforcement half of those rules **does not ship
in this edition**. `nina.toml` cites it at `tests/policy/nina-review-role.json`, which is a
path in `mozart-orchestration`, not here — this repo has no `tests/` tree. Fetch it from
upstream before granting live-read mode; a role nobody adapted is not an enforced one.

Landed separately and first: the **HEAVY trigger now classifies access-control changes
at any layer**, not just Kubernetes RBAC. An identity-plane change previously
classified STANDARD and skipped the pre-flight gate entirely. That fix is independent
of nina and survives a revert of the rest.

### Added — conductor self-verification: mozart's own derived claims get a control, a linkage, and a lint

Ported from `mozart-orchestration`'s conductor-self-verification campaign. mozart's own conclusions
— a check it ran, a dispute it settled, a fact it copied into a brief — now carry the same discipline
M2/M7 already demand of everyone else's checks.

1. **The conductor record** (`## Conductor record`, new state-file section in `SKILL.md`'s template)
   — one row per derived claim (`check` | `adjudication` | `fact`), with a linked gate/finding/
   correction id and a control that could have shown the claim false. A ticked row-required gate key
   (DELIVER `5 9 10 13 P<N>`, OPERATE `1:fact 4 6`, INCIDENT `1 5`) needs a linked row; the section
   may stay empty only while no such obligation exists.
2. **Dispute handling when mozart is a party** — settled by a third source (a command neither side
   wrote) in a linked `adjudication` row, or escalated to the operator or a fresh, unanchored dick.
   A design judgment no command could settle is dispositioned **`rejected (judgment)`** with a
   decisions-log citation; **`rejected (user)`** remains the user-overruled case. Both are new
   findings-ledger dispositions alongside `fixed`/`rejected`/`accepted-risk`.
3. **A decisions log** (`<slug>.decisions.md`): every judgment call gets a decision, reasoning,
   bounds, and a revisit trigger.
4. **The mutation manifest** — one field (or a coupled set with a stated rationale) per OPERATE/
   INCIDENT mutation, with literal `ignore:` field paths for what a read-back may skip and
   `<redacted>` for secret-bearing values. Landed in `SKILL.md`'s OPERATE/INCIDENT sections,
   `hank.toml`'s Apply step, and `otto.toml`'s change-plan bullet.
5. **`scripts/mozart-lint.sh` and `scripts/mozart-metrics.sh` now read both artifact roots**
   (`.mozart/` and `thoughts/shared/`), matching `mozart-orchestration`'s dual-root support — a
   target repo following either convention lints and aggregates cleanly.
6. **Lint Checks K and L** (`conductor-missing`, `conductor-unlinked`, `conductor-row`,
   `conductor-reference`, `decision-trigger`, `mutation-manifest`). `MOZART_LINT_CONDUCTOR_SINCE`
   overrides the adoption-date constant as a fixture test hook; every run that sets it prints
   `conductor adoption date overridden: <value>` before any finding, so the override can never be
   silent. **Check L shares Check K's PD1 adoption boundary** rather than carrying a second copy of
   it, so a pre-adoption campaign's change-ledger rows are never flagged — including rows written
   before the manifest column existed — while a post-adoption campaign gets no grandfathering.
7. **Check J** (`missing-2b`, DELIVER-family campaigns missing their `2b. Constraints` row) is new
   to this port — it fires only when the Flow field resolves to the DELIVER family, or is
   unparseable and the file has stage rows `2.`, `3.` and `12.`.
8. **Pre-existing drift fixed alongside**: `SKILL.md`'s state-file template gains the
   `## Degraded controls` section it was missing; `jackson.toml` gains the mid-build M2/M7 bullet it
   was missing; `SKILL.md`'s per-phase gate bullet is restored to name the plan's Automated commands
   convention (`tagged (phase N)`) and its secret-scan bullet regains the empty-input liveness
   sentences, matching source. Restructuring the linter onto source's dual-root shape also brought
   source's current Check D/E stage-letter handling (a `12b` duplicate now reports as `12b`, not
   `12`) and its `## Paths`-scoped Check G, both of which this port had predated.
9. `scripts/mozart-metrics.sh` gains a `== conductor ==` block: campaigns carrying a conductor record
   (and how many are exempt), conductor rows by kind, controlled check/adjudication rows, unverified
   facts, and the wrong-override rate (rejected findings later reversed, with a `rejected (judgment)`
   share). It also fixes the same findings/escapes placeholder-detection bug as source: a literal `<`
   anywhere in the line used to skip real findings, not just template placeholder cells.

**Reconciliation round 1** (mirrored from source's external pre-merge review, F47-F52). Checks K/L
claimed the same current+legacy scope as Checks C/D and did not have it: the legacy prefixless flat
glob (`plans/<date>-<slug>.state.md`) was missing, and the adoption date was read off the raw
basename, so every `active-`/`finished-` prefixed file classified as pre-adoption regardless of its
date — a post-adoption state file with no `## Conductor record` linted clean in either layout.
Conductor and change-ledger rows were split on a raw `|`, so a `source` cell holding a shell pipeline
shifted every later cell and an **empty control parsed as filled**; `SKILL.md` said "no literal pipe"
and nothing enforced it. Both tables now honour `\|` as an escaped pipe and reject any row whose cell
count disagrees with its header (`conductor-row` / `mutation-manifest`), with the finding deferred to
`END` so it stays behind PD1's adoption gate. `mozart-metrics.sh` applies the same rule and prints
the count of rows it skipped rather than tallying a shifted row as controlled. Roots and file lists
were whitespace-delimited strings in `mozart-metrics.sh` and Check F word-split an unquoted
`$(find ...)`, so a checkout under a path containing a space reported "no state files" and a stale
campaign there went unreported; both are NUL-delimited arrays now. Finally, PD1's adoption gate has
two limbs — slug date on or after the cutoff, **or** a header already present — and `decision-trigger`
implemented only the first, so a campaign carrying a conductor record with a pre-cutoff slug date had
its rows checked while its decisions log went unchecked. Parity with source is proven by
`scripts/check-field-note-parity.py`'s `behaviour` subcommand against source's fixture corpus.

### Changed

- `.codex/skills/mozart/SKILL.md` drops the "An unattended run needs a decision log" field note —
  promoted into the decisions-log mechanism it described (item 3 above). mozart's field-note count
  goes from three to two; jackson's is unchanged. The 0.1.0 entry below, which records that note
  arriving, is history and is left as written.

**Known gap, disclosed rather than silently ignored**: this port does not implement source's
`missing-12b` check (Check I). codex has no stage 12b — it lacks upstream commit `e9232c0` (the
`.mozart/` artifact-root convention as a persona-level fallback, worktree isolation, and hank's
version-resolution step). Porting Check I unmodified would flag every codex DELIVER campaign for a
stage this port doesn't run. Both gaps are scoped as their own follow-up campaign; the scripts above
already read both artifact roots so that campaign has less to change.

## [0.1.0] - 2026-09-13

### Added (field-notes harvest, 2026-09-13)
- Four prose field notes ported from the unmerged `learnings/2026-09-09-mozart-local-field-notes`
  branch of mozart-orchestration: jackson gains one entry (mutation testing finds missing tests, not
  weak ones); mozart's `SKILL.md` gains three (state known-wrong facts in the brief; scope empirical
  findings to platform/version/date; an unattended run needs a decision log).
- Three findings from the same branch are installed as **procedural mechanisms** rather than prose,
  because prose contracts inside a persona were shown not to change behavior: M2 (verify the
  measuring instrument against a known-FAIL case before trusting it) and M7 (every counting/globbed
  check needs a population floor and a named member, to rule out vacuity and coincidence) are added
  to `harry.toml`'s Verification section; M4 (after a plan revision, name the pre-revision sections a
  mechanism touches) is added to `SKILL.md`'s stage-6 Iterate procedure.
- `harry.toml` also receives the minimal 7-line Automated/Manual convention subset (the two-list
  shape plus the binding clause) so M2/M7 attach to a real field rather than referencing one that
  doesn't exist in this port's Verification section. Porting the full 24-line convention is out of
  scope for this change.
- **This port has no gate suite and no CI** (`.github` does not exist here), so all three mechanisms
  are enforced procedurally only — by the agent reading and following them — not by any automated
  check. Cross-port agreement with mozart-orchestration and mozart-copilot is verified by hand with
  `check-field-note-parity.py`, run from mozart-orchestration against all three worktrees pre-merge.

### Fixed (2026-09-12: capability-vs-claim parity)
- **Personas no longer promise an action their `sandbox_mode` can't perform.** Repaired 14
  instances of one defect class across three shapes, prose-only — **no `sandbox_mode` value
  changed anywhere**; the capability model stays byte-identical, **12 `read-only` / 8
  `workspace-write`**, at base and head.
- **Task-class**: harry and jackson stop claiming they spawn sub-agents. `agents.max_depth = 1`
  means a subagent (which is what harry and jackson run as) can't spawn one of its own — harry's
  Design It Twice now drafts multiple radically-different interface proposals himself, in one
  pass, and jackson's mid-build specialist routing gets the matching repair.
- **Field-note class (8 instances)**: `dexter`, `dick`, `ian`, `librarian`, `otto`, `sarah`,
  `valerie`, `xander` — all `read-only` — stop claiming they self-append to `LEARNINGS.md` via a
  `Bash` heredoc. `read-only` blocks every filesystem write, heredoc included. `LEARNINGS.md`'s
  append protocol is rewritten: the 12-name `read-only` population is spelled out explicitly (was
  a stale 3-name list), and each of the 8 personas now returns its proposed entry to mozart, who
  appends on its behalf.
- **Write-class artifacts (3 instances)**: dick's findings doc, sarah's brief, and otto's OPERATE
  change plan are returned to mozart rather than written by the `read-only` persona claiming (or
  implying) authorship. New `## Persisting artifacts for read-only agents` section in the mozart
  skill grants and bounds mozart's authorship (verbatim — no condensing, no editorializing);
  mozart's own editable-file allowlist amended to match. Six stale tool-noun references
  (`**Bash**` → `**shell**`, `**WebFetch**` → `**web fetch**`) fixed on dick and tessa. xander's
  unhedged "check advisories" imperative is reordered, not weakened — web fetch leads, and the
  four scanners (`npm audit`, `pip-audit`, `govulncheck`, `cargo audit`) stay in the same clause,
  now gated behind it rather than claimed unconditionally under a `read-only` persona.
- **Two of this campaign's own verification checks corrected mid-flight**: the persist-clause
  regex was widened from matching one accidental emphasis form to verifying the underlying
  property regardless of bold/italic/plain; the constraint-card producer registry was corrected
  to name all four eligible producers (`xander`/`ian` via stage `2b`; `xander`/`ian`/`librarian`/
  `otto` via a stage-3 consult) instead of the two `2b` alone can trigger.
- Ticket: [#1](https://github.com/jstuart0/mozart-codex/issues/1). Plan:
  `.mozart/plans/active/2026-09-12-deliver-capability-claim-parity.md`. Commits: `6a32d74`,
  `8f19be4`, `6363bcf`, `dd65e4e`, `9e8244c`.

**What this doesn't establish.** The premise — that `read-only` actually blocks a write — was
verified empirically, but the result is narrower than it looks: a depth-0 `read-only` session was
denied with *"patch rejected: writing is blocked by read-only sandbox; rejected by user approval
settings,"* a **conjunctive** denial, and the probe ran under `approval: never`. The demonstrated
claim is *"read-only + never-approve denies,"* not *"read-only denies"* on its own. The artifact
enumeration behind this repair is closed only over path-shaped references in this repo's
`.md`/`.toml`/`.sh`/`.example` text — file-terminated paths and bare directory references on
write-verb lines — and does not cover an artifact named only in prose with no path token, a path
computed at runtime, or file types outside that list. Whether `~/.mozart/evals/` — which lies
outside any workspace root — is writable is left open; it's a session-sandbox question
`config.toml.example` doesn't answer, and resolving it was out of scope for a prose-only campaign.
This port still has no contract-gate suite and no CI; every check above was run by hand, and
nothing mechanically prevents regression.

### Added (parity sync with mozart-orchestration, 2026-09-11: pre-plan specialist consults + stage `2b`)
- **Stage `2b` (Constraints)** — a narrow, bounded constraint pushed automatically before harry
  drafts the plan, when the task itself trips one of two triggers: who-may-do-what (an
  authorization/trust-boundary question) → xander; a change that could falsify a guarantee this
  repo already publishes → ian. Returns a ≤5-bullet `must`/`must-not` constraint card, persisted to
  `thoughts/shared/plans/active/<slug>.constraints.md`. Off by default; an untriggered `2b` costs
  exactly **four** touches (a state-file `## Stage progress` row, a flow-sketch `## Stage trace`
  line, a state-file `Paths: Constraints` line reading `n/a`, and one clause in the intake
  rationale) and nothing else.
- **`## Consult requested`** — a new return type for harry: before a plan exists, he can pull in
  one of four lenses (xander, ian, librarian, or otto) with exactly one bounded question, stating
  what he assumes if declined. Capped at 2 consults per campaign (`Consult count` in the state
  file); hitting the cap surfaces to the user and tells harry to draft against his own fallback
  rather than stall.
- **Anchoring carve-out extended**: a lens that supplied a constraint card (at `2b` or via a
  consult) is re-invoked at stage 4 as a fresh spawn, never a continued thread — the same reviewer
  never meets its own prior conclusion already anchored in the plan under review.
- Ported across 10 conductor sites (`.codex/skills/mozart/SKILL.md`, `PIPELINE.md`) and 11 persona
  `DELIVER stages:` lines. No new agent; no `sandbox_mode` change.
- Ticket: [#1](https://github.com/jstuart0/mozart-codex/issues/1). Commits: `46576b3`, `27b1f53`.

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
