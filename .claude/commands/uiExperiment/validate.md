---
description: Audit the UI Lab project against CLAUDE.md — Latest-badge uniqueness, hero count accuracy, required sidebar sections, link integrity, freeze rule, changelog ordering — and report a pass/fail checklist.
argument-hint: [optional: experiment-name to scope the audit]
---

# /uiExperiment:validate — Audit the project against CLAUDE.md

You are the **Validator** (run as a Sonnet-class audit). Read `CLAUDE.md` IN FULL first. Perform a READ-ONLY audit, produce a checklist with explicit PASS / FAIL per item, then OFFER to fix any failures (do not fix until the user approves).

Optional scope from `$ARGUMENTS`: if an experiment name is given, focus the per-file checks on that family; always run the global checks.

## Checks to perform

### 1. Latest badge uniqueness (experiments.html)
- Count the actual Latest **element**, not the raw `badge-updated` token. The token appears twice in a clean repo — once in the CSS rule (`.badge-updated{...}`) and once in the span — so a naive token count yields 2 and reports a false FAIL on a clean repo (verified: 2 token occurrences, 1 element). Instead match the element, e.g. `grep -oE 'class="badge badge-updated"' experiments.html | wc -l` (or match `badge-updated">Latest`).
- PASS iff EXACTLY one element matches. Report the enclosing card's `<h3>`; FAIL if zero or more than one (list all holders).

### 2. Hero stat counts match reality
- Read `.hero .stats` counts: experiments, versions, and (if present) origination series.
- Compute actuals from the filesystem:
  - **Versions** = count of `*-v{N}.html` files.
  - **Experiments** = distinct families that have a card in `experiments.html` (the declared source of truth). This MUST equal the distinct families present on disk — enforced by the orphan check below — so a passing orphan check makes the two definitions interchangeable. An origination family (one with a base `{family}.html` origin page) counts as 1 experiment.
  - **Origination series** = count of base `{family}.html` origin pages (files with no `-v{N}` suffix, excluding `experiments.html`, `changelog.html`, `index.html`).
- Print **stated vs actual** for all three hero stats.
- PASS iff each stat equals its actual.

### 2b. Orphan-family detection (experiments.html is the source of truth)
`experiments.html` is the declared source of truth but nothing currently checks that every on-disk family actually has a card. (Verified drift example: `dual-sidebar-v1.html` exists and is logged in `changelog.html` yet has zero references in `experiments.html`, and the hero stated 22 experiments while 23 `-vN` families exist on disk.)
- Derive **families on disk**: `ls *-v[0-9]*.html | sed -E 's/-v[0-9]+\.html$//' | sort -u`.
- Derive **families referenced by cards** in `experiments.html` (from card `href`s, normalized to the same family prefix).
- Compute the **symmetric difference** and FAIL on any mismatch, listing both groups explicitly:
  - **on disk but not indexed** (orphan families with no card), and
  - **indexed but not on disk** (cards pointing at families with no files).
- PASS iff the two sets are identical.

> NOTE: Do NOT auto-fix the underlying orphan or change the hero numbers as part of this audit. Registering a missing family (e.g. `dual-sidebar`) edits `experiments.html` and is a separate data-correction action that must be surfaced to the user as its own remediation — see the offer-to-fix step at the end. This check only detects and reports the drift.

### 3. Required sidebar sections (every experiment + variant file)
The "every file MUST have an Agent Trace + exp-model badge" rules apply **prospectively**: they are enforced on files created after those conventions were introduced, and are **waived for files frozen before the rule existed**. Roughly ~125 older frozen files predate the Agent Trace / `exp-model` conventions and can never be brought into compliance (the freeze rule forbids editing them), so hard-failing them would make the validator output untrustworthy. Split the requirement by era/role:

**(a) Hard-require on ALL files** (any era):
- **Purpose** section, **Prompt** section, the `← All Experiments` back link to `experiments.html`, and a version badge (`exp-version` for standard, `exp-variant` for origination variant).

**(b) Hard-require on the LATEST file of each family AND on any file created after the convention's introduction:**
- **Agent Trace** section and an `exp-model` badge (in addition to everything in (a)).
- `exp-autonomous` present iff the file is autonomous.
- **Versions** card.
- Agent Trace: each Builder/Researcher/Validator entry has a duration badge and a full-prompt toggle; `at-prompt-{N}` IDs are unique within the file; the `input → label → panel` sibling order is intact.
- **Provenance (prospective files only):** the file contains an `exp-runmeta` row expressing all four provenance axes as structured chips — **model** (`exp-model`, required), **effort** (`exp-effort` with a `high`|`medium`|`low` modifier, required), **harness** (`exp-harness`, **warn** if missing), and **autonomy** (`exp-autonomy` with a `collaborative`|`autonomous` modifier, **warn** if missing). Each Agent Trace entry carries an `exp-at-effort` chip. Model + effort missing is a FAIL; harness or autonomy missing is a WARN (not a hard FAIL).

**(c) Grandfathered frozen legacy files** that lack Agent Trace / `exp-model` (and therefore the provenance chips): emit an informational **"grandfathered"** note rather than a FAIL. Never FAIL a frozen pre-convention file for missing Agent Trace, `exp-model`, or provenance chips.

Report PASS/FAIL per file for (a) and (b), plus a count of grandfathered files (list offenders for real failures; summarize if many pass).

### 4. Internal link integrity
- Collect every internal `href` in `experiments.html`, `changelog.html`, `index.html`, and each experiment sidebar (back links, `view-latest`, `exp-vc-entry`, `exp-vc-origin`, changelog targets).
- PASS iff every linked local `.html` file exists on disk. List any broken links (source file → missing target).

### 5. Freeze rule respected
For each experiment family, frozen files are every version except the latest. The rule allows frozen files to differ from a pristine baseline ONLY in their sidebar `.exp-versions-card` (the sole permitted edit).
- **Honest limitation:** the repo currently has only ~2 commits and no per-file creation history, so there is no pristine per-file baseline to diff against. A `git diff`/`git log` mechanism therefore CANNOT detect pre-import edits, and there is no defined baseline for a "heuristic fallback." When no per-file baseline exists, this check MUST report **`CANNOT VERIFY (no per-file baseline available)`** rather than silently PASSing.
- **What it CAN do today (structural fallback):** confirm by structural inspection that each frozen file's only diff-prone region is the `.exp-versions-card` — i.e. the Versions card is the sole region expected to vary across siblings, and the surrounding sidebar content (purpose/prompt/agent-trace) is internally consistent and well-formed. Report any frozen file whose structure suggests out-of-card mutation as a concrete violation; otherwise report the structural inspection result alongside the `CANNOT VERIFY` status for the diff-based mechanism.
- **Recommended future work (do NOT implement here):** check in a content-hash manifest of each file's non-Versions-card sidebar region, so freeze compliance can be verified mechanically against a stable baseline.
- Status: `CANNOT VERIFY` for the diff mechanism (until a baseline exists) + PASS/violations for the structural inspection. Do not emit a bare PASS that implies the freeze was mechanically verified.

### 6. Versions-card completeness
- For each family, the LATEST file's Versions card must list every sibling version newest-first with the current one marked `current`. Each FROZEN file's Versions card must also include the newest version (the backfill rule). PASS iff complete; list gaps.

### 7. Changelog ordering & integrity (changelog.html)
Changelog entries use the `M/D h:mmam/pm` format with **no year**, so timestamp monotonicity is not mechanically checkable — a legitimate year boundary (e.g. Dec→Jan) reads as a decrease and is indistinguishable from a real ordering bug. Drop the timestamp-monotonicity claim and verify only the structurally checkable invariants:
- The `<!-- Append new entries on top. Newest first. -->` marker comment is present.
- New/topmost entries sit **immediately after** that marker comment.
- Every `gc-meta` entry has a non-empty `.gc-prompt-inner`.
- PASS/FAIL with any anomalies.

> Optional future work (do NOT implement here): add a machine-readable `data-ts` attribute to each `.gc-entry` so chronological ordering can be verified mechanically.

### 7b. Changelog provenance completeness (changelog.html)
Every **post-framework** changelog entry — both file-creation `<a>` entries and `gc-meta` entries — must carry the provenance chips. Legacy **pre-framework** entries are exempt: use the reframe entry as the cutoff (entries at or before the reframe are pre-framework and exempt; entries after it are prospective and required to comply).
- **Required (FAIL if missing):** `gc-model` chip and `gc-effort` chip on every post-cutoff entry.
- **Recommended (WARN if missing):** `gc-harness` chip (text `Claude Code`) and `gc-autonomy` chip (with a `collaborative`|`autonomous` modifier) — these complete the four-axis provenance model (model · effort · harness · autonomy).
- PASS iff every post-cutoff entry carries `gc-model` + `gc-effort`. FAIL listing offenders (entry timestamp + target). Separately, WARN-list any post-cutoff entry missing `gc-harness` or `gc-autonomy`.

### 8. Pure HTML/CSS rule (spot check)
- Flag any `<script>` tag or inline JS handler (`on*=`) in experiment files. PASS iff none found.

## Output

Print a checklist:

```
UI Lab Validation
[PASS] 1.  Latest badge uniqueness — held by {card}
[FAIL] 2.  Hero counts — experiments stated 22, actual 23
[FAIL] 2b. Orphan families — on disk but not indexed: dual-sidebar
[PASS] 3.  Required sidebar sections — N pass, M grandfathered
[----] 5.  Freeze rule — CANNOT VERIFY (no per-file baseline); structural inspection PASS
[PASS] 7.  Changelog ordering & integrity
[FAIL] 7b. Changelog provenance completeness — 2 post-framework entries missing gc-effort
[WARN] 7b. Changelog provenance — 1 entry missing gc-harness/gc-autonomy (recommended)
...
```

Use `[----]` (or `CANNOT VERIFY`) for checks that cannot be mechanically confirmed; do not pretend they PASS. Use `[WARN]` for recommended-but-not-required provenance chips (harness, autonomy) that are missing — these do not fail the audit but should be surfaced.

For every FAIL, give a one-line remediation. Then ASK the user whether to fix the failures. If they approve, fix them following the relevant `CLAUDE.md` rules (and respect the freeze rule — only Versions-card edits to frozen files), then log a `gc-meta` changelog entry (per `/uiExperiment:changelog`) describing the fixes with provenance chips (Model, Effort — ASK if unknown, Harness=Claude Code, Autonomy).

**Surface data-correction actions separately:** any orphan-family registration (Check 2b) or hero-count edit (Check 2) modifies `experiments.html` and is a distinct data-correction action — present it as its own remediation, do not bundle it silently with cosmetic fixes.
