---
description: Append a meta/structural entry to changelog.html for a manual change (CLAUDE.md edits, refactors, fixes) with the verbatim prompt and provenance chips preserved.
argument-hint: ["verbatim prompt"] [files affected]
---

# /uiExperiment:changelog — Log a meta/structural change

You are the **Lead** agent. Read the "Updating changelog.html" section of `CLAUDE.md` first. This command records any change that does NOT create a new experiment/version file — CLAUDE.md updates, restructuring experiments.html, prompt-driven fixes to existing files, framework/tooling changes. (For new files use `/uiExperiment:new`, `/uiExperiment:version`, or `/uiExperiment:origination`, which write their own creation entries.)

User input (may be empty): `$ARGUMENTS`

## Step 0 — Gather inputs

Ensure you have:

1. **Verbatim prompt** — the EXACT prompt text that triggered this change, copy-pasted (typos and all), never paraphrased. This is the primary value of the entry. If not supplied in `$ARGUMENTS`, use the user's actual request message verbatim, or ASK for it.
2. **Files affected** — comma-separated list (e.g. `claude.md, experiments.html`).
3. **Short description** — one human-readable line of what changed (for `.gc-desc`).
4. **Run metadata (provenance — mandatory, all FOUR axes):** Every entry must record all four provenance axes as structured chips:
   - **Model** → `gc-model` class (`opus`/`sonnet`/`haiku`).
   - **Effort** → `gc-effort` modifier (`high`/`medium`/`low`). **ASK if you cannot infer it** — do not guess.
   - **Harness** → `gc-harness` chip, text `Claude Code`.
   - **Autonomy** → `gc-autonomy` chip with modifier `collaborative` or `autonomous`. **ASK if you cannot determine it** — a change with any human prompt/guidance in-session is `collaborative`; only fully agent-driven changes are `autonomous`.

## Step 1 — Build the entry

Use the Meta/Improvement Entry Template from `CLAUDE.md`, extended with ALL FOUR provenance chips:

```html
<div class="gc-entry gc-meta">
  <span class="gc-time">{M/D h:mmam/pm}</span>
  <span class="gc-action">Updated</span>
  <span class="gc-target">{files affected, comma-separated}</span>
  <span class="gc-model {opus|sonnet|haiku}">{Model}</span>
  <span class="gc-effort {high|medium|low}">{Effort}</span>
  <span class="gc-harness">Claude Code</span>
  <span class="gc-autonomy {collaborative|autonomous}">{collaborative|autonomous}</span>
  <span class="gc-prompt-badge">Prompt ↓</span>
  <span class="gc-desc">{Short human-readable description}</span>
  <div class="gc-prompt"><div class="gc-prompt-inner">
    <div class="gc-prompt-label">Prompt</div>
    {VERBATIM prompt. Use <br><br> between paragraphs.}
  </div></div>
</div>
```

For file-creation `<a>` entries, carry the same four chips inline before the `gc-target` (e.g. `gc-model`, `gc-effort`, `gc-harness`, `gc-autonomy`, then `gc-target`).

- `gc-action` is `Updated` for modifications (`Created` is only for new-file `<a>` entries).
- Timestamp = current time, format `M/D h:mmam/pm` (e.g. `2/1 9:03am`).
- All four provenance chips (`gc-model`, `gc-effort`, `gc-harness`, `gc-autonomy`) are REQUIRED on every new entry so the changelog dogfoods the full provenance model. The verbatim prompt is preserved in `.gc-prompt-inner` and must never be paraphrased.
- If `changelog.html` lacks CSS for any chip class (`.gc-effort` + `.high`/`.medium`/`.low`, `.gc-harness`, or `.gc-autonomy` + `.collaborative`/`.autonomous`), add minimal styling consistent with the existing chip classes (small mono rounded chip, color-coded; autonomy tinted with `--auto` amber when autonomous) so the chip renders — and log THIS structural CSS addition as part of the affected files.

## Step 2 — Insert at the top

Insert the entry at the very top of `.changelog-entries`, immediately after the `<!-- Append new entries on top. Newest first. -->` comment. NEVER remove or reorder existing entries — the log is immutable.

## Step 3 — Report

Confirm the entry added (time, files, description) and that all four provenance chips were recorded (model · effort · harness · autonomy). Quote the first line of the verbatim prompt back so the user can confirm it was preserved exactly.
