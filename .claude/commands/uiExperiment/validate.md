---
description: Audit the UI Lab project against CLAUDE.md — Latest-badge uniqueness, hero count accuracy, required sidebar sections, link integrity, freeze rule, changelog ordering — and report a pass/fail checklist.
argument-hint: [optional: experiment-name to scope the audit]
---

# /uiExperiment:validate — Audit the project against CLAUDE.md

You are the **Validator** (run as a Sonnet-class audit). Read `CLAUDE.md` IN FULL first. Perform a READ-ONLY audit, produce a checklist with explicit PASS / FAIL per item, then OFFER to fix any failures (do not fix until the user approves).

Optional scope from `$ARGUMENTS`: if an experiment name is given, focus the per-file checks on that family; always run the global checks.

## Checks to perform

### 1. Latest badge uniqueness (experiments.html)
- Count `badge-updated` ("Latest") occurrences in `experiments.html`. PASS iff EXACTLY one. Report which card holds it; FAIL if zero or more than one (list all holders).

### 2. Hero stat counts match reality
- Read `.hero .stats` counts: experiments, versions, and (if present) origination series.
- Compute actuals from the filesystem:
  - **Versions** = count of `*-v{N}.html` files.
  - **Experiments** = count of distinct experiment families (distinct `{name}` prefixes among `*-v*.html`), where an origination family (one with a base `{family}.html` origin page) counts as 1 experiment.
  - **Origination series** = count of base `{family}.html` origin pages (files with no `-v{N}` suffix, excluding `experiments.html`, `changelog.html`, `index.html`).
- PASS iff each stat equals its actual. Report stated vs actual for each.

### 3. Required sidebar sections (every experiment + variant file)
For each `*-v{N}.html` (and each origin `{family}.html`), verify the sidebar contains:
- **Purpose** section, **Prompt** section, **Agent Trace** section, **Versions** card.
- Correct badge: `exp-version` (standard) or `exp-variant` (origination variant); an `exp-model` badge; `exp-autonomous` present iff the file is autonomous.
- The `← All Experiments` back link to `experiments.html`.
- Agent Trace: each Builder/Researcher/Validator entry has a duration badge and a full-prompt toggle; `at-prompt-{N}` IDs are unique within the file; the `input → label → panel` sibling order is intact.
Report PASS/FAIL per file (list offenders; summarize if many pass).

### 4. Internal link integrity
- Collect every internal `href` in `experiments.html`, `changelog.html`, `index.html`, and each experiment sidebar (back links, `view-latest`, `exp-vc-entry`, `exp-vc-origin`, changelog targets).
- PASS iff every linked local `.html` file exists on disk. List any broken links (source file → missing target).

### 5. Freeze rule respected
- For each experiment family, frozen files are every version except the latest. Verify frozen files differ from a pristine baseline ONLY in their sidebar `.exp-versions-card` (the sole permitted edit). Use git where available (`git diff`/`git log` on the file) to detect edits outside the Versions card; if git is unavailable, heuristically check that each frozen file's Versions card lists all current siblings and flag any file whose non-Versions sidebar content (prompt/purpose/agent-trace) appears to have been altered after creation.
- PASS iff no frozen file shows edits outside its Versions card. List violations.

### 6. Versions-card completeness
- For each family, the LATEST file's Versions card must list every sibling version newest-first with the current one marked `current`. Each FROZEN file's Versions card must also include the newest version (the backfill rule). PASS iff complete; list gaps.

### 7. Changelog ordering & integrity (changelog.html)
- Newest entries on top, immediately after the `<!-- Append new entries on top. Newest first. -->` comment.
- Every `gc-meta` entry has a non-empty `.gc-prompt-inner`. Spot-check that timestamps are non-increasing top→bottom. PASS/FAIL with any anomalies.

### 8. Pure HTML/CSS rule (spot check)
- Flag any `<script>` tag or inline JS handler (`on*=`) in experiment files. PASS iff none found.

## Output

Print a checklist:

```
UI Lab Validation
[PASS] 1. Latest badge uniqueness — held by {card}
[FAIL] 2. Hero counts — versions stated 142, actual 143
...
```

For every FAIL, give a one-line remediation. Then ASK the user whether to fix the failures. If they approve, fix them following the relevant `CLAUDE.md` rules (and respect the freeze rule — only Versions-card edits to frozen files), then log a `gc-meta` changelog entry (per `/uiExperiment:changelog`) describing the fixes with provenance chips (Model, Effort — ASK if unknown, Harness=Claude Code, Autonomy).
