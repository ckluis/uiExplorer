---
description: Create the next version of an existing experiment — copy the latest file, make changes, sync the Versions card across ALL prior versions, and update experiments.html + changelog.html.
argument-hint: [experiment-name] [what to change]
---

# /uiExperiment:version — New version of an existing experiment

You are the **Lead** agent. Read `CLAUDE.md` IN FULL first — implement "When Creating a New Version" and "Updating experiments.html — Adding a New Version" EXACTLY. This command automates the error-prone bookkeeping (Versions-card backfill, Latest-badge move, count bumps).

User input (may be empty): `$ARGUMENTS`

## Step 0 — Gather inputs

Parse `$ARGUMENTS` for the experiment name and the requested change. Then ensure you have:

1. **Experiment name** — kebab-case (e.g. `command-palette`). Detect the highest existing `{name}-v{N}.html` to determine the latest version `N` and the new version `N+1`. List the files you find so the user can confirm.
2. **Change description / verbatim prompt** — the EXACT prompt that produces this version's changes from the previous one. Copy-paste, never paraphrase. If only a name was given, ask for the literal prompt or treat the user's request message as the verbatim prompt.
3. **Updated purpose** (if scope changed) — otherwise carry forward.
4. **Run metadata (provenance — mandatory):** Model, Effort (`high`/`medium`/`low` — **ASK if you cannot infer it**), Harness = `Claude Code`, Autonomy (collaborative vs autonomous).

Confirm: copying `{name}-v{N}.html` → `{name}-v{N+1}.html`.

## Step 1 — Copy the latest file

Copy `{name}-v{N}.html` to `{name}-v{N+1}.html`. (Do not start from the template — start from the latest version so the experiment content carries forward.) Verify the new filename does not already exist.

## Step 2 — Make the changes (Builder = Opus)

Spawn a **Builder** sub-agent with `model: opus` to apply the requested changes to `{name}-v{N+1}.html`'s experiment content. Constraints: pure HTML/CSS, no JS, no external deps beyond Google Fonts, standalone-renderable. It must NOT touch the sidebar. Record its start/end times and full verbatim prompt for the Agent Trace.

## Step 3 — Update the new file's sidebar (Sidebar = Sonnet)

Spawn a **Sidebar** sub-agent with `model: sonnet` to update `{name}-v{N+1}.html`'s sidebar:
- Version badge → `v{N+1}`; correct `.exp-model` badge; autonomy badge only if autonomous (with the autonomous note prepended to the Prompt section).
- **Prompt** section → the VERBATIM change prompt (`<br><br>` between paragraphs) plus an `.exp-ask` block per AskUserQuestion interaction.
- **Purpose** → updated if scope changed.
- **Agent Trace** → entries for every agent this session (Lead, Builder, Sidebar, etc.) with durations, hover timestamps, model badges, and full-prompt toggles (`at-prompt-{N}` unique IDs) for Builder/Researcher/Validator.
- **Versions** card → list ALL versions `v1..v{N+1}` newest-first; `v{N+1}` gets `class="...current"`; each entry shows its model indicator.

## Step 4 — Backfill the Versions card in ALL previous version files (the ONLY permitted frozen-file edit)

For EACH of `{name}-v1.html` through `{name}-v{N}.html`, add the new `v{N+1}` entry to the top of its sidebar `.exp-versions-card`, newest-first. This is mandatory and is the ONLY change allowed to frozen files — do NOT alter their experiment content, prompts, purpose, agent trace, or anything else. (For an origination series, also keep the `exp-vc-origin` link(s) at the top.) Verify each edit changed only the Versions card.

## Step 5 — Update experiments.html ("Adding a New Version")

Follow `CLAUDE.md` exactly:
1. Increment the **version count** in `.hero .stats` (experiment count unchanged).
2. Move the single `badge-updated` Latest badge to this experiment's card; ensure exactly one exists.
3. Bump the card's `badge-versions` span (e.g. "5 versions" → "6 versions").
4. Update the card description if scope changed.
5. Add a new changelog entry as the FIRST `<a>` inside the card's `.changelog`, using the Changelog Entry Template, with `class="changelog-entry latest"`.
6. Remove `latest` from the previously-top entry (→ `class="changelog-entry"`).
7. Update the card's "View latest" link to `{name}-v{N+1}.html`.
8. Autonomy: if autonomous, add `class="...autonomous"` on the new entry's `.changelog-desc` (and the card's `badge-autonomous` if not already present).

## Step 6 — Log in changelog.html (creation entry)

Append a **file-creation** entry at the very top of `.changelog-entries` (after the newest-first comment) for `{name}-v{N+1}.html`, using the File Creation Entry Template with the file's birth time, `Created`, model class/name. Never reorder existing entries. (The frozen-file Versions-card backfill is captured implicitly by this version creation — no separate meta entry is required unless the user asks.)

## Step 7 — Report

Summarize: new file, N→N+1, how many prior files had their Versions card backfilled, hero version count before→after, Latest-badge move, and the changelog entry. Note provenance (model / effort / harness / autonomy).
