---
description: Scaffold a brand-new UI Lab experiment (v1) end-to-end — build the HTML/CSS, document the sidebar, register it in experiments.html, and log it in changelog.html.
argument-hint: [experiment-name] [one-line purpose]
---

# /uiExperiment:new — Create a new experiment (v1)

You are the **Lead** agent of the UI Lab team (see the Team System in `CLAUDE.md`). Read `CLAUDE.md` IN FULL before doing anything — it is the source of truth for every rule below. Faithfully implement it; do not improvise file naming, badge placement, or counts.

User input (may be empty): `$ARGUMENTS`

## Step 0 — Gather inputs

Parse `$ARGUMENTS` for an experiment name and/or purpose. Then make sure you have ALL of the following. If any is missing, ASK the user (use AskUserQuestion where there are clear choices):

1. **Name** — kebab-case, no version suffix (e.g. `command-palette`). Validate: lowercase, hyphen-separated, no spaces. The file will be `{name}-v1.html`. Reject a bare `{name}.html` target — that base name is reserved for origination files only.
2. **Purpose** — one or two sentences: what this explores and why it matters.
3. **Verbatim prompt** — the EXACT prompt text that is driving this experiment. This must be copy-pasted, never paraphrased. If the user only gave you a name/purpose, ask them for the literal prompt, or treat their actual request message as the verbatim prompt and record it as such.
4. **Run metadata (provenance — mandatory):**
   - **Model** — which Claude model is producing the content (e.g. "Opus 4.6"). Used for the `opus`/`sonnet`/`haiku` badge classes.
   - **Effort level** — the agent's `/effort` reasoning setting: `high`, `medium`, or `low`. If you cannot infer it from context, **ASK the user** — do not guess.
   - **Harness** — `Claude Code`.
   - **Autonomy** — `collaborative` (any human prompt/guidance) or `autonomous` (no human interaction during creation). If there was any human prompt, it is collaborative.
5. **Device preview?** — Ask whether the experiment has meaningful responsive behavior worth the device-preview switcher. If yes, use the Device Preview Switcher pattern from `CLAUDE.md`.

Confirm the resolved name, purpose, and `{name}-v1.html` target before writing.

## Step 1 — Scaffold the file

Verify `{name}-v1.html` does not already exist. Copy `templates/experiment-template.html` to `{name}-v1.html`. Fill every placeholder in the template (name, version `v1`, model badge, purpose, prompt, etc.). If the template is missing, fall back to constructing the file from the Standard Experiment Sidebar CSS + HTML patterns in `CLAUDE.md`.

## Step 2 — Build the experiment content (Builder = Opus)

Spawn a **Builder** sub-agent via the Agent/Task tool with `model: opus`. Its job: produce the pure HTML/CSS experiment content that lives inside `.experiment-content` (or inside `.window` if device preview is used). Constraints to pass it:
- Pure HTML/CSS only. NO JavaScript. No external deps beyond Google Fonts.
- Must render standalone in a browser.
- Honor the device-preview decision from Step 0.
- Do NOT touch the sidebar — that's the Sidebar agent's job.

Record the Builder's start time, end time, and the FULL verbatim prompt you give it (for the Agent Trace).

## Step 3 — Document the sidebar (Sidebar = Sonnet)

Spawn a **Sidebar** sub-agent via the Agent/Task tool with `model: sonnet`. Its job: populate the experiment info sidebar per `CLAUDE.md`:
- Experiment name, `v1` version badge, correct `.exp-model {class}` badge.
- Autonomy badge `<span class="exp-autonomous">Fully Autonomous</span>` ONLY if autonomy = autonomous (and prepend the autonomous note to the Prompt section).
- **Purpose** section.
- **Prompt** section — the VERBATIM prompt, `<br><br>` between paragraphs. Include an `.exp-ask` block for every AskUserQuestion interaction that occurred (question, all options, selected option, answer text).
- **Agent Trace** — one entry per agent that ran (Lead, Builder, Sidebar, and any Researcher/Validator). Each entry: role, duration badge, model badge, action, hover timestamps, and a full-prompt toggle (`at-prompt-{N}`, unique sequential IDs) containing the complete verbatim prompt for Builder/Researcher/Validator. Workflow one-liner in `.exp-at-workflow` (e.g. `Lead → Builder → Sidebar`).
- **Versions** card — just the `v1` current entry with its model indicator.

Pass the Sidebar agent: the model, effort, harness, autonomy, every agent's start/end times and verbatim prompts, and all AskUserQuestion transcripts.

## Step 4 — Register in experiments.html ("Adding a New Experiment")

Follow `CLAUDE.md` → "Updating experiments.html — Adding a New Experiment" EXACTLY:
1. Increment BOTH the experiment count and the version count in `.hero .stats`.
2. Move the single `<span class="badge badge-updated">Latest</span>` — remove it from whichever card currently holds it, add it to the new card. Exactly one Latest badge must exist afterward.
3. Add a new `.card` using the New Experiment Card Template. The `v1` changelog entry inside the card gets `class="changelog-entry latest"`; point the `view-latest` link at `{name}-v1.html`.
4. If autonomy = autonomous, add `<span class="badge badge-autonomous">Autonomous</span>` to `.card-meta` and `class="...autonomous"` on the entry's `.changelog-desc`.

## Step 5 — Log in changelog.html (creation entry)

Append a **file-creation** entry at the very top of `.changelog-entries` (immediately after the `<!-- Append new entries on top. Newest first. -->` comment), using the File Creation Entry Template with the file's birth time, `Created`, the model class/name, and `{name}-v1.html`. Never reorder existing entries.

## Step 6 — Report

Summarize: file created, hero counts before→after, where the Latest badge moved from/to, and the changelog entry added. Note the recorded provenance (model / effort / harness / autonomy).

**Do not** modify any frozen experiment file. Only `{name}-v1.html`, `experiments.html`, and `changelog.html` may change.
