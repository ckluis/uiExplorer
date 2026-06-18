---
description: Run an origination series — create an origin page from a verbatim master prompt, then fan out N parallel Builder agents to produce autonomous variants, and register it all in experiments.html + changelog.html.
argument-hint: [family-name] [N variants] [research direction]
---

# /uiExperiment:origination — Origination series (parallel variants)

You are the **Lead** agent. Read `CLAUDE.md` IN FULL first and implement the **Origination Prompt Pattern** EXACTLY. The non-negotiable rule applies: **nothing is more important than documenting the prompts** — the verbatim master prompt on the origin page, and the exact per-variant build prompt in each variant sidebar. Summaries are never acceptable.

User input (may be empty): `$ARGUMENTS`

## Step 0 — Gather inputs

Parse `$ARGUMENTS` for the family name, variant count, and direction. Then ensure you have:

1. **Family name** — kebab-case (e.g. `crm-dashboard`). The origin page is `{family}.html` (the reserved base name); variants are `{family}-v1.html` … `{family}-v{N}.html`.
2. **Verbatim master/origination prompt** — the EXACT text the user typed, copy-pasted including typos and informal phrasing. This is the single most important artifact. Use `<br><br>` between paragraphs when rendering.
3. **N** — number of variants (typically 5–10).
4. **AskUserQuestion interactions** — if you use AskUserQuestion to refine direction, capture each: question, all options, selected option (or Other), and the user's answer text. These go on the origin page AND inform the build prompts.
5. **Run metadata (provenance — mandatory):** Model(s), Effort (`high`/`medium`/`low` — **ASK if you cannot infer it**), Harness = `Claude Code`, Autonomy = `autonomous` for variants (each variant is built autonomously by a sub-agent and gets the "Fully Autonomous" badge).

## Step 1 — Research (Researcher = Sonnet, optional but recommended)

Spawn a **Researcher** sub-agent with `model: sonnet` to gather reusable patterns and shape the N distinct variant directions (each a different interpretation/persona/UI approach). Record its start/end and full prompt for the trace.

## Step 2 — Build the origin page `{family}.html`

Create `{family}.html` using the standard sidebar pattern but with an **"Origin"** badge (not a version badge). Content area must include:
- The **verbatim master prompt** (`<br><br>` between paragraphs).
- Every **AskUserQuestion** interaction, rendered with the AskUserQuestion visualization pattern.
- Research domains / context.
- A grid/list of all N variants with one-line descriptions.
- Execution-model note: research agent → N parallel build agents → validation agent.

If `templates/origin-template.html` exists, base the origin page on it.

Fill the origin page `exp-runmeta` chips (`exp-effort`/`exp-harness`) with the actual recorded provenance; the origin template ships a hardcoded effort value — replace it with the real effort/class. Verify no placeholders or stale defaults remain.

## Step 3 — Fan out N parallel Builders ([Builder×N], model: opus)

Spawn N **Builder** sub-agents IN PARALLEL (one message, multiple Agent/Task tool calls), each `model: opus`, each producing one `{family}-v{K}.html` from `templates/origin-template.html` (fallback: standard variant pattern). For EACH variant agent:
- Craft an EXACT build prompt (variant name, tagline, core metaphor, layout, key UI elements, color/mood, any role-toggle behavior). SAVE this exact prompt verbatim — it must appear in that variant's sidebar Prompt section.
- Constraints: pure HTML/CSS, no JS, no deps beyond Google Fonts, standalone-renderable.
- Each variant uses the **Origination Variant Sidebar** pattern: green `exp-variant` badge, always-present `exp-autonomous` badge, **Variant Focus** section, **Prompt** section prefixed with *"This version was generated autonomously by a sub-agent. The prompt below is the exact prompt given to the build agent…"* followed by the FULL build prompt, an **Origination Prompt** section linking to `{family}.html`, and a **Versions** card listing the `exp-vc-origin` link plus all sibling variants (current one marked `current`).
- Include the Agent Trace in each variant sidebar (Lead → Researcher → Builder → Sidebar/Validator) with durations, timestamps, and full-prompt toggles.
- Fill each variant sidebar's `exp-runmeta` row (`exp-effort`/`exp-harness`) with the actual recorded provenance; the variant template ships a hardcoded effort value — replace it with the real effort/class. Verify no placeholders or stale defaults remain.
- Populate the per-agent `exp-at-effort` chips in every Agent Trace entry with that agent's actual effort — do not leave the template defaults.

Record start/end times for each parallel Builder.

## Step 4 — Validate (Validator = Sonnet)

Spawn a **Validator** sub-agent with `model: sonnet` to verify all N variants against `CLAUDE.md`: links resolve, sidebars complete, prompts verbatim, exactly the right badges. Record its trace.

## Step 5 — Register in experiments.html (Origination Series Card)

Follow `CLAUDE.md` → "experiments.html — Origination Series Card":
1. Add the origination CSS only if not already present (`.badge-origination`, `.card-origination`).
2. Add ONE `.card card-origination` using the Origination Card HTML Template: `badge-origination` "Origination Series", `badge-versions` "{N} variants", one changelog entry per variant (each `.changelog-desc` gets `autonomous`), and a "View origination prompt →" link to `{family}.html`.
3. Move the single `badge-updated` Latest badge to this card (exactly one Latest in the file).
4. **Hero stats:** experiment count +1, version count +N, origination-series count +1 (add the third stat span if missing, per the Hero Stats Update template).
5. For multiple waves on one family, use the inline `changelog-origination-divider` pattern and `#origination-{N}` anchors.

## Step 6 — Log in changelog.html

Append at the very top of `.changelog-entries`, newest-first, NEVER reordering existing entries:
- One **meta** entry (`gc-meta`) for the origination prompt itself — VERBATIM master prompt in `.gc-prompt-inner` (`<br><br>` between paragraphs), plus provenance chips (`gc-model`, `gc-effort {high|medium|low}`).
- One **file-creation** entry per variant (`{family}-v1.html` … `{family}-v{N}.html`) and one for the origin page `{family}.html`.

## Step 7 — Report

Summarize: origin page + N variant files created, hero stats before→after (all three counts), Latest-badge move, changelog entries added (1 meta + N+1 creations). Confirm every variant's verbatim build prompt was preserved. Note provenance (model / effort / harness / autonomy=autonomous).
