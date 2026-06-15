# UI Explorations — Experiment Framework

This folder contains a collection of UI experiments. Each experiment is a self-contained HTML/CSS file that can be opened directly in a browser. All experiments are linked together through `experiments.html`.

## The Framework Is the Product

This repo is a **forkable framework** called **The UI Lab**, not just a personal gallery. The experiments here are the *showcase* — proof the system works — but the point is that **anyone can fork it and run their own lab**. The framework is hostable on GitHub Pages and driven by a `/uiExperiment` command suite. Keep this framing in mind: changes should make the system more usable by a stranger who forks it, not just prettier for the owner.

**Three shell pages (the chrome around experiments):**
- `index.html` — **The landing page and GitHub Pages home.** Explains the concept, the provenance model, and how to fork + run your own lab. This is what visitors see first. Built in the **"Lab Instrument"** design language (mono accents `IBM Plex Mono` + `Inter`, fine grid lines, cyan `#2ee6c6` accent on near-black `#0a0c10`, shared CSS `:root` tokens). When stats change, update its hero `.stats` counts too.
- `experiments.html` — The example gallery / **source of truth for the experiment list**. Restyled to Lab Instrument; reframed as "the example lab."
- `changelog.html` — The immutable build log. Restyled to Lab Instrument.

**`/uiExperiment` command suite** (`.claude/commands/uiExperiment/`): `new`, `version`, `origination`, `changelog`, `validate`, `help`. These automate the bookkeeping the rest of this file describes (index updates, prior-version Versions cards, changelog entries, hero counts). Prefer running a command over doing the steps by hand. The long-form rules below are the spec the commands implement.

**Templates** (`templates/`): `experiment-template.html` and `origin-template.html` carry the **new Lab Instrument** experiment sidebar + device-preview shell. New experiments are scaffolded from these — they are the canonical, up-to-date implementation of the sidebar/preview/agent-trace patterns documented later in this file. **If a pattern's CSS here ever disagrees with the template, the template wins.**

**The freeze + the shell redesign.** The Lab Instrument redesign applies to the three shell pages and to *future* experiments (via the templates). The 137 existing experiment files keep their original inline sidebar/preview CSS — that is archive integrity, not an oversight. Never restyle a frozen experiment file. (See the freeze rule under "When Creating a New Version.")

**Provenance run-metadata (required).** Every experiment is documented not just with its prompt and model, but with the **run settings** that produced it:
- **Model** — `.exp-model`/`.gc-model` badge (`opus`/`sonnet`/`haiku`).
- **Effort** — the `/effort` reasoning level (high/medium/low). Use the `.exp-effort` (sidebar/agent-trace) and `.gc-effort` (changelog) chip with a `high`/`medium`/`low` modifier class. If you don't know the effort, **ask**.
- **Harness** — e.g. "Claude Code" (`.exp-harness`).
- **Autonomy** — collaborative (default) vs `Fully Autonomous`.
Record these on changelog entries, agent-trace entries, and version metadata. The principle: *every change must show what caused it — prompt, model, and the settings it ran under.*

**Hosting.** `README.md` is the repo front door; `.nojekyll` + `.github/workflows/deploy-pages.yml` deploy the root to GitHub Pages (or use Settings → Pages → deploy from `main`/root). Pure HTML/CSS means the static root *is* the site.

## Rules

### File Structure
- `index.html` — **Landing page / GitHub Pages home.** Explains the framework; links into the gallery. Lab Instrument design.
- `experiments.html` — Central index of all experiments. **Source of truth** for the experiment list.
- `changelog.html` — Chronological log of all actions. Linked from experiments.html footer and the nav.
- `README.md` — Repo front door (markdown mirror of `index.html`); fork + hosting instructions.
- `templates/` — `experiment-template.html`, `origin-template.html`. Canonical Lab Instrument shells new experiments are scaffolded from.
- `.claude/commands/uiExperiment/` — The `/uiExperiment` command suite (`new`, `version`, `origination`, `changelog`, `validate`, `help`).
- `.github/workflows/deploy-pages.yml`, `.nojekyll` — GitHub Pages deploy.
- `{experiment-name}.html` — Origination file (only if experiment is an origination series)
- `{experiment-name}-v1.html` — Version 1 of an experiment
- `{experiment-name}-v2.html` — Version 2, and so on (`-v3`, `-v4`, etc.)
- `claude.md` — This file. The rules.

**Important:** The base name `{experiment-name}.html` (without version suffix) is reserved exclusively for origination files. All versions — including v1 — always use the `-v{N}` suffix. Origination variants also use `-v{N}` (e.g., `-v1` through `-v10`), not ordinal suffixes like `-01`.

### Every Experiment File Must

1. **Be pure HTML/CSS.** No JavaScript. No external dependencies beyond Google Fonts. Each file must work when opened standalone in a browser.

2. **Include an experiment info sidebar** on the left side (~280px wide, always visible). This sidebar contains:
   - **Experiment name** (e.g., "Platform Sidebar")
   - **Version** (e.g., "v1", "v2")
   - **Model** — Which Claude model produced this version (e.g., "Opus 4.6", "Sonnet 4.5"). Use the `.exp-model` badge with the appropriate model class (`opus`, `sonnet`, `haiku`).
   - **Run metadata** — The settings the version ran under: **Effort** (the `/effort` reasoning level — high/medium/low, via the `.exp-effort` chip with a matching modifier class) and **Harness** (e.g. "Claude Code", via `.exp-harness`). If the effort level is unknown, ask. See "Provenance run-metadata" above. The `templates/experiment-template.html` already includes this row.
   - **Purpose** — What this experiment explores and why it matters
   - **Prompt** — The **exact prompt** (copy-pasted) that was used to generate or change this version. For v1, this is the original prompt. For v2+, this is the exact prompt that produced the changes from the previous version. Do not paraphrase or summarize — use the literal prompt text. Preserve paragraph breaks using `<br><br>` between paragraphs so the prompt is readable in the browser (HTML collapses whitespace without explicit breaks). If the experiment was developed through multiple iterative prompts before this framework existed, note that instead.
   - **AskUserQuestion interactions** — When the AskUserQuestion tool is used during a session, include each interaction inline in the prompt section using the structured `.exp-ask` block (see pattern below). Show: the question asked, all options presented, which option was selected (or "Other"), and the user's answer text. This preserves the full conversational context that led to the final implementation.
   - **Agent Trace** — Documents every agent spawned during creation: role, model, duration, start/end timestamps, action summary, and the full prompt given to each agent (viewable via click-to-expand toggle). See the Agent Trace Pattern section below.
   - **Previous version link** (if applicable) — e.g., `platform-sidebar-v1.html` from `platform-sidebar-v2.html`
   - **"← All Experiments" link** — Always links to `experiments.html`

3. **Use the standard sidebar CSS pattern** documented below.

4. **Include an autonomy indicator** when applicable. If the experiment or version was generated in a fully autonomous agent session — meaning no human interaction occurred during creation (no AskUserQuestion exchanges, no iterative human prompts, no mid-session guidance) — it must be flagged:
   - Add `<span class="exp-autonomous">Fully Autonomous</span>` next to the `.exp-version` badge
   - In the Prompt section, prepend the note: *"This version was generated autonomously. The prompt shown is a summary of the agent's intent, not a verbatim human prompt."*
   - The prompt section should still attempt to capture the directive or intent that launched the session, but it is understood to be approximate rather than a literal copy-paste
   - If a human was involved at any point (even just the initial prompt with follow-up AskUserQuestion), do **not** mark it as autonomous — that is the default collaborative mode

### When Creating a New Version

1. Copy the latest version file and rename it with the next version number
2. Make your changes to the new file
3. Update the experiment info sidebar with the new version number, prompt, and purpose
4. Add the new version to `experiments.html`
5. **Update the Versions card in ALL previous version files** to include the new version entry. This is mandatory — every older version (v1, v2, etc.) must list the new version in its sidebar Versions card. This is the only permitted modification to frozen files — experiment content, prompts, and purpose must never change.
6. **Add an entry to `changelog.html`** — Every new version file must be logged as a creation entry at the top of the changelog.

**Important: Log everything.** Every action that changes any file in this project — creating versions, updating CLAUDE.md, fixing previous version sidebars, structural changes — must get a changelog entry. Use a file-creation entry (`<a>`) for new files, and a meta entry (`<div class="gc-meta">`) with the verbatim prompt for modifications. The changelog is the immutable record of how this system gets built.

### When Creating a New Experiment

1. Create `{experiment-name}-v1.html` as v1
2. Add the experiment info sidebar with all required fields
3. Add the experiment to `experiments.html`

### Naming Convention

Use kebab-case for experiment names: `platform-sidebar-v1.html`, `data-table-v1.html`, `command-palette-v1.html`. The base name without a version suffix (e.g., `crm-dashboard.html`) is reserved for origination files only.

### Updating experiments.html — Adding a New Version

When adding a new version of an existing experiment, update `experiments.html` with these steps:

1. **Update hero stats** — Increment the version count in `.hero .stats` (e.g., "6 versions" → "7 versions")
2. **Move the `Latest` badge** — Remove `<span class="badge badge-updated">Latest</span>` from whichever card currently has it, and add it to this card's `.card-meta`. Only one card should have this badge at a time — the most recently updated experiment.
3. **Update the card's version badge** — Increment the `badge-versions` span (e.g., "5 versions" → "6 versions")
4. **Update the card description** if the new version meaningfully changes scope
5. **Add a new changelog entry** as the first `<a>` inside the card's `.changelog` div, before existing entries
6. **Move the `latest` class** — Add `class="changelog-entry latest"` to the new entry; remove `latest` from the previous top entry (change it to just `class="changelog-entry"`)
7. **Update the "View latest" link** at the bottom of the changelog to point to the new version file

#### Changelog Entry Template

```html
<a href="{experiment-name}-v{N}.html" class="changelog-entry latest">
  <span class="changelog-v">v{N}</span>
  <span class="changelog-time">{M/DD h:mmam/pm}</span>
  <span class="changelog-desc">{Short one-line description}</span>
  <span class="changelog-model {model-class}">{Model}</span>
  <span class="changelog-detail">{Detailed description of changes, 1-2 sentences}</span>
</a>
```

Model classes: `opus` (gold), `sonnet` (purple), `haiku` (cyan). Display short name: "Opus", "Sonnet", "Haiku".

### Updating experiments.html — Adding a New Experiment

When adding an entirely new experiment, update `experiments.html` with these steps:

1. **Update hero stats** — Increment both the experiment count and version count in `.hero .stats`
2. **Move the `Latest` badge** — Remove `<span class="badge badge-updated">Latest</span>` from whichever card currently has it, and include it in the new card. Only one card should have this badge at a time — the most recently created or updated experiment.
3. **Add a new `.card` div** inside the `.grid` div (no CSS changes needed — accent colors are auto-assigned via `nth-child`)

#### New Experiment Card Template

```html
<!-- Experiment Name -->
<div class="card">
  <div class="card-accent"></div>
  <div class="card-body">
    <div class="card-meta">
      <h3>{Experiment Name}</h3>
      <span class="badge badge-updated">Latest</span>
      <span class="badge badge-versions">1 version</span>
    </div>
    <p class="desc">{Brief description of what the experiment explores.}</p>
    <div class="tags">
      <span class="tag">{tag1}</span>
      <span class="tag">{tag2}</span>
    </div>
    <div class="changelog">
      <a href="{experiment-name}.html" class="changelog-entry latest">
        <span class="changelog-v">v1</span>
        <span class="changelog-time">{M/DD h:mmam/pm}</span>
        <span class="changelog-desc">Initial — {short description}</span>
        <span class="changelog-model {model-class}">{Model}</span>
        <span class="changelog-detail">{Detailed description, 1-2 sentences.}</span>
      </a>
      <a href="{experiment-name}.html" class="view-latest">View latest →</a>
    </div>
  </div>
</div>
```

#### Autonomy in experiments.html

When a card represents an experiment where the latest version (or any version) was fully autonomous, add `<span class="badge badge-autonomous">Autonomous</span>` to the `.card-meta`. For changelog entries of autonomous versions, add `class="autonomous"` to the entry's `.changelog-desc` span — this renders it in amber to visually distinguish agent-only work from collaborative work.

Note: No per-experiment CSS is needed. The `.card-accent` bar automatically gets a gradient color from a rotating palette of 8 colors via `nth-child` selectors. The masonry layout (`column-count: 3`) handles varying card heights naturally — experiments with many versions create taller cards.

### Updating changelog.html

`changelog.html` is a standalone page that serves as the chronological record of every action taken across all experiments — file creations, version additions, structural changes, and meta improvements. It is linked from the footer of `experiments.html`.

**Rules:**
- **Append on top only.** New entries always go at the top of `.changelog-entries`, immediately after the `<!-- Append new entries on top. Newest first. -->` comment.
- **One entry per action.** Every file creation, version addition, or significant modification gets its own entry.
- **Use file creation birth time** as the timestamp for new files. For meta/structural changes, use the current time.
- **Never remove or reorder existing entries.** The log is immutable once written.
- **Every change gets logged.** This includes meta changes like updating CLAUDE.md, restructuring experiments.html, or any prompt-driven improvement that doesn't create a new experiment file.

**When to add entries:**
- Creating a new experiment file (v1)
- Creating a new version of an existing experiment
- Creating an origination series (one entry per variant + one for the origin page)
- Any structural or meta change (e.g., updating CLAUDE.md, refactoring experiments.html)
- Any prompt-driven improvement to existing files

#### File Creation Entry Template

For new files, use an `<a>` tag linking to the file:

```html
<a href="{file}.html" class="gc-entry">
  <span class="gc-time">{M/D h:mmam/pm}</span>
  <span class="gc-action">Created</span>
  <span class="gc-model {model-class}">{Model}</span>
  <span class="gc-target">{filename.html}</span>
</a>
```

Model classes: `opus` (gold), `sonnet` (purple), `haiku` (cyan). Display short name: "Opus", "Sonnet", "Haiku".

#### Meta/Improvement Entry Template

For changes that don't create a new experiment file (structural changes, CLAUDE.md updates, prompt-driven improvements), use a `<div>` with the `gc-meta` class. The prompt is shown in an expandable panel that reveals on hover.

```html
<div class="gc-entry gc-meta">
  <span class="gc-time">{M/D h:mmam/pm}</span>
  <span class="gc-action">Updated</span>
  <span class="gc-target">{files affected, comma-separated}</span>
  <span class="gc-prompt-badge">Prompt ↓</span>
  <span class="gc-desc">{Short human-readable description of what changed}</span>
  <div class="gc-prompt"><div class="gc-prompt-inner">
    <div class="gc-prompt-label">Prompt</div>
    {The exact verbatim prompt that triggered this change. Use <br><br> between paragraphs.}
  </div></div>
</div>
```

**Key elements:**
- `.gc-prompt-badge` — A visible "Prompt ↓" badge that signals hover capability. Lights up on hover.
- `.gc-prompt` — Expandable container that smoothly reveals the prompt text on hover (CSS `max-height` transition).
- `.gc-prompt-inner` — Styled panel with border and background for readability.
- Left border accent — Meta entries have a purple left border that brightens on hover.

**Action values:** `Created` for new files, `Updated` for modifications to existing files.

**Timestamp format:** `M/D h:mmam/pm` (e.g., `2/1 9:03am`).

**Prompt preservation:** The `.gc-prompt-inner` content is mandatory and must contain the user's exact prompt text. This is the primary value of the changelog — it documents *what was asked* alongside *what changed*.

---

## Standard Experiment Sidebar CSS Pattern

Copy this into every experiment file. The experiment content sits to the right of this sidebar.

```css
/* ─── Experiment Info Sidebar ─── */
.experiment-frame {
  display: flex;
  height: 100vh;
  overflow: hidden;
}

.experiment-sidebar {
  width: 280px;
  min-width: 280px;
  background: #1a1a2e;
  color: #e0e0e0;
  padding: 24px 20px;
  overflow-y: auto;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  font-size: 13px;
  line-height: 1.5;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.experiment-sidebar a {
  color: #8b9cf7;
  text-decoration: none;
}

.experiment-sidebar a:hover {
  text-decoration: underline;
}

.experiment-sidebar .exp-back {
  font-size: 12px;
  opacity: 0.7;
  display: block;
  margin-bottom: 8px;
}

.experiment-sidebar h1 {
  font-size: 18px;
  font-weight: 700;
  color: #fff;
  margin: 0;
}

.experiment-sidebar .exp-version {
  display: inline-block;
  background: rgba(139, 156, 247, 0.15);
  color: #8b9cf7;
  font-size: 11px;
  font-weight: 600;
  padding: 2px 8px;
  border-radius: 4px;
  margin-top: 4px;
}

.experiment-sidebar .exp-section {
  border-top: 1px solid rgba(255,255,255,0.08);
  padding-top: 16px;
}

.experiment-sidebar .exp-section h2 {
  font-size: 10px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: #888;
  margin: 0 0 8px 0;
}

.experiment-sidebar .exp-section p {
  margin: 0;
  color: #ccc;
}

.experiment-sidebar .exp-prompt {
  background: rgba(255,255,255,0.04);
  border-radius: 6px;
  padding: 12px;
  font-size: 12px;
  color: #bbb;
  font-style: italic;
}

.experiment-sidebar .exp-prev {
  font-size: 12px;
}

.experiment-sidebar .exp-autonomous {
  display: inline-block;
  background: rgba(243, 156, 18, 0.15);
  color: #f39c12;
  font-size: 10px;
  font-weight: 600;
  padding: 2px 8px;
  border-radius: 4px;
  margin-top: 4px;
  margin-left: 6px;
}

/* ─── Model Badge ─── */
.experiment-sidebar .exp-model {
  display: inline-block;
  font-size: 10px;
  font-weight: 600;
  padding: 2px 8px;
  border-radius: 4px;
  margin-top: 4px;
  margin-left: 6px;
}
.experiment-sidebar .exp-model.opus {
  background: rgba(229, 166, 59, 0.15);
  color: #e5a63b;
}
.experiment-sidebar .exp-model.sonnet {
  background: rgba(139, 156, 247, 0.15);
  color: #8b9cf7;
}
.experiment-sidebar .exp-model.haiku {
  background: rgba(94, 196, 212, 0.15);
  color: #5ec4d4;
}

.experiment-content {
  flex: 1;
  overflow: auto;
  position: relative;
}
```

## Standard Experiment Sidebar HTML Pattern

```html
<div class="experiment-frame">
  <aside class="experiment-sidebar">
    <div>
      <a href="experiments.html" class="exp-back">← All Experiments</a>
      <h1>Experiment Name</h1>
      <span class="exp-version">v1</span>
      <span class="exp-model opus">Opus 4.6</span>
      <!-- Add only if fully autonomous: -->
      <!-- <span class="exp-autonomous">Fully Autonomous</span> -->
    </div>

    <div class="exp-section">
      <h2>Purpose</h2>
      <p>What this experiment explores and why.</p>
    </div>

    <div class="exp-section">
      <h2>Prompt</h2>
      <div class="exp-prompt">
        The prompt or description of changes that produced this version.
      </div>
    </div>

    <!-- Agent Trace — always include, documents all agents used during creation -->
    <div class="exp-section">
      <h2>Agent Trace</h2>
      <div class="exp-at-workflow">Lead → Builder → Sidebar</div>
      <div class="exp-agent-trace">
        <!-- See Agent Trace Pattern section below -->
      </div>
    </div>

    <!-- Versions card — always include, shows all versions of this experiment -->
    <div class="exp-section">
      <h2>Versions</h2>
      <div class="exp-versions-card">
        <!-- For origination series only: -->
        <!-- <a href="{experiment-name}.html" class="exp-vc-origin">Origination Prompt →</a> -->
        <!-- Most recent version first, current version has class="current" -->
        <a href="experiment-name-v2.html" class="exp-vc-entry">
          <span class="exp-vc-v">v2</span>
          <span class="exp-vc-desc">Short description</span>
        </a>
        <a href="experiment-name-v1.html" class="exp-vc-entry current">
          <span class="exp-vc-v">v1</span>
          <span class="exp-vc-desc">Initial version</span>
        </a>
      </div>
    </div>
  </aside>

  <div class="experiment-content">
    <!-- Your experiment content goes here -->
  </div>
</div>
```

## AskUserQuestion Sidebar CSS Pattern

Add this CSS to any experiment file that includes AskUserQuestion interactions in the prompt section.

```css
/* ─── AskUserQuestion Visualization ─── */
.experiment-sidebar .exp-ask {
  background: rgba(139, 156, 247, 0.08);
  border: 1px solid rgba(139, 156, 247, 0.2);
  border-radius: 6px;
  padding: 10px 12px;
  margin-top: 10px;
  font-style: normal;
}
.experiment-sidebar .exp-ask-label {
  font-size: 9px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  color: #8b9cf7;
  margin-bottom: 6px;
}
.experiment-sidebar .exp-ask-question {
  font-size: 11px;
  color: #ddd;
  font-style: normal;
  margin-bottom: 8px;
  line-height: 1.4;
}
.experiment-sidebar .exp-ask-options {
  display: flex;
  flex-direction: column;
  gap: 4px;
  margin-bottom: 8px;
}
.experiment-sidebar .exp-ask-option {
  font-size: 10px;
  color: #999;
  padding: 3px 8px;
  background: rgba(255,255,255,0.04);
  border-radius: 4px;
  border: 1px solid rgba(255,255,255,0.06);
}
.experiment-sidebar .exp-ask-option.selected {
  color: #fff;
  background: rgba(139, 156, 247, 0.18);
  border-color: rgba(139, 156, 247, 0.4);
  font-weight: 600;
}
.experiment-sidebar .exp-ask-answer {
  font-size: 11px;
  color: #ccc;
  font-style: normal;
  padding: 6px 8px;
  background: rgba(46, 204, 113, 0.08);
  border: 1px solid rgba(46, 204, 113, 0.2);
  border-radius: 4px;
  line-height: 1.4;
}
.experiment-sidebar .exp-ask-answer strong {
  color: #2ecc71;
  font-weight: 600;
}
```

## AskUserQuestion Sidebar HTML Pattern

Place this inside the `.exp-prompt` div, after the prompt text, for each AskUserQuestion interaction that occurred.

```html
<div class="exp-ask">
  <div class="exp-ask-label">AskUserQuestion</div>
  <div class="exp-ask-question">The question that was asked?</div>
  <div class="exp-ask-options">
    <div class="exp-ask-option">Option 1 — description</div>
    <div class="exp-ask-option">Option 2 — description</div>
    <div class="exp-ask-option selected">Other (selected)</div>
  </div>
  <div class="exp-ask-answer"><strong>Answer:</strong> The user's exact response text.</div>
</div>
```

- Mark the chosen option with the `selected` class
- If the user picked a named option, mark that option as `selected` instead of "Other"
- If there were multiple AskUserQuestion interactions, include one `.exp-ask` block for each, in order

---

## Device Preview Switcher Pattern

Experiments can include a monitor/mobile preview switcher that wraps the experiment content in a "virtual device" window. This allows viewers to see how the experiment looks at different screen sizes without resizing their browser.

### How It Works
- The `.experiment-content` area gets a gradient background and centers a `.window` element
- A preview bar with desktop/mobile toggle icons sits above the window
- Radio buttons control which mode is active
- Mobile mode shrinks the window to 375×720px and triggers responsive CSS adjustments
- All state is driven by CSS radio buttons — no JavaScript

### When to Use
Use the device preview switcher when the experiment has meaningful responsive behavior worth demonstrating (e.g., navigation patterns, dashboards, layouts with multiple breakpoints). Skip it for simple experiments where responsive behavior isn't relevant.

### Device Preview CSS Pattern

Add this CSS to experiments that use the device preview switcher.

```css
/* ─── Device Preview Switcher ─── */
.experiment-content {
  flex: 1;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}

.preview-bar {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  padding: 10px 0 8px;
}

.preview-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 36px;
  height: 36px;
  border-radius: 8px;
  cursor: pointer;
  color: rgba(255,255,255,0.5);
  transition: background 0.2s, color 0.2s;
}

.preview-btn:hover {
  color: rgba(255,255,255,0.8);
  background: rgba(255,255,255,0.1);
}

.preview-btn svg { width: 20px; height: 20px; }

#preview-web:checked ~ .preview-bar label[for="preview-web"],
#preview-mobile:checked ~ .preview-bar label[for="preview-mobile"],
#preview-full:checked ~ .preview-bar label[for="preview-full"] {
  background: rgba(255,255,255,0.2);
  color: white;
}

.window {
  width: 1360px;
  height: 820px;
  background: var(--bg, #f5f4f1);
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 25px 60px rgba(0,0,0,0.35), 0 0 0 0.5px rgba(0,0,0,0.1);
  position: relative;
  transition: width 0.4s cubic-bezier(0.4, 0, 0.2, 1),
              height 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

/* Mobile preview mode */
#preview-mobile:checked ~ .window {
  width: 375px;
  height: 720px;
}

/* Fullscreen preview mode */
#preview-full:checked ~ .preview-bar {
  position: absolute;
  top: 8px;
  z-index: 100;
  background: rgba(0,0,0,0.4);
  border-radius: 10px;
  padding: 4px;
}

#preview-full:checked ~ .window {
  width: 100%;
  height: 100%;
  border-radius: 0;
  box-shadow: none;
}
```

### Device Preview HTML Pattern

Place the radio inputs and preview bar **inside** `.experiment-content`, before the `.window` element. The preview radios must be siblings of (or ancestors of selectors targeting) `.preview-bar` and `.window`.

```html
<div class="experiment-content">
  <!-- Device preview state -->
  <input type="radio" name="preview" id="preview-web" checked>
  <input type="radio" name="preview" id="preview-mobile">
  <input type="radio" name="preview" id="preview-full">

  <div class="preview-bar">
    <label for="preview-web" class="preview-btn" title="Desktop view">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
        <rect x="2" y="3" width="20" height="14" rx="2"/>
        <line x1="8" y1="21" x2="16" y2="21"/>
        <line x1="12" y1="17" x2="12" y2="21"/>
      </svg>
    </label>
    <label for="preview-mobile" class="preview-btn" title="Mobile view">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
        <rect x="5" y="2" width="14" height="20" rx="2"/>
        <line x1="12" y1="18" x2="12" y2="18" stroke-width="2"/>
      </svg>
    </label>
    <label for="preview-full" class="preview-btn" title="Fullscreen view">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
        <polyline points="15 3 21 3 21 9"/>
        <polyline points="9 21 3 21 3 15"/>
        <line x1="21" y1="3" x2="14" y2="10"/>
        <line x1="3" y1="21" x2="10" y2="14"/>
      </svg>
    </label>
  </div>

  <div class="window">
    <!-- Your experiment content goes here -->
  </div>
</div>
```

Add mobile-specific CSS overrides for your experiment using the `#preview-mobile:checked ~ .window` selector pattern to adapt layouts, font sizes, and spacing for the 375px viewport. The fullscreen mode expands the window to fill the entire `.experiment-content` area, with the preview bar floating over the content.

---

## Sidebar Versions Card Pattern

Every experiment file includes a Versions section in the sidebar that lists all versions of the experiment. The current version is highlighted. For origination series, a link to the origination prompt appears at the top.

### How It Works
- All versions are listed in reverse chronological order (most recent first)
- The current version gets the `current` class
- Each entry has a version number and short description
- For origination series, an origination prompt link appears above the version list
- Only the **current version** needs this section — past frozen files are never modified

### Sidebar Versions Card CSS Pattern

```css
/* ─── Sidebar Versions Card ─── */
.experiment-sidebar .exp-versions-card {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.experiment-sidebar .exp-vc-origin {
  display: block;
  font-size: 11px;
  font-weight: 600;
  color: #2ecc71;
  text-decoration: none;
  padding: 6px 10px;
  background: rgba(46, 204, 113, 0.08);
  border: 1px solid rgba(46, 204, 113, 0.15);
  border-radius: 6px;
  margin-bottom: 6px;
}

.experiment-sidebar .exp-vc-origin:hover {
  background: rgba(46, 204, 113, 0.15);
  text-decoration: none;
}

.experiment-sidebar .exp-vc-entry {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 5px 10px;
  border-radius: 5px;
  text-decoration: none;
  color: inherit;
  transition: background 0.15s;
}

.experiment-sidebar .exp-vc-entry:hover {
  background: rgba(255,255,255,0.06);
  text-decoration: none;
}

.experiment-sidebar .exp-vc-entry.current {
  background: rgba(139, 156, 247, 0.12);
  border: 1px solid rgba(139, 156, 247, 0.25);
}

.experiment-sidebar .exp-vc-v {
  font-size: 11px;
  font-weight: 700;
  color: #8b9cf7;
  min-width: 24px;
}

.experiment-sidebar .exp-vc-entry.current .exp-vc-v {
  color: #fff;
}

.experiment-sidebar .exp-vc-desc {
  font-size: 11px;
  color: #999;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.experiment-sidebar .exp-vc-entry.current .exp-vc-desc {
  color: #ccc;
}

/* ─── Model Indicator in Versions Card ─── */
.experiment-sidebar .exp-vc-model {
  font-size: 9px;
  font-weight: 600;
  padding: 1px 5px;
  border-radius: 3px;
  margin-left: auto;
  flex-shrink: 0;
}
.experiment-sidebar .exp-vc-model.opus {
  background: rgba(229, 166, 59, 0.12);
  color: #e5a63b;
}
.experiment-sidebar .exp-vc-model.sonnet {
  background: rgba(139, 156, 247, 0.12);
  color: #8b9cf7;
}
.experiment-sidebar .exp-vc-model.haiku {
  background: rgba(94, 196, 212, 0.12);
  color: #5ec4d4;
}
```

### Sidebar Versions Card HTML Pattern

```html
<div class="exp-section">
  <h2>Versions</h2>
  <div class="exp-versions-card">
    <!-- For origination series only: -->
    <!-- <a href="{experiment-name}.html" class="exp-vc-origin">Origination Prompt →</a> -->
    <!-- Most recent version first, current version has class="current" -->
    <a href="...-v5.html" class="exp-vc-entry current">
      <span class="exp-vc-v">v5</span>
      <span class="exp-vc-desc">Short description of this version</span>
      <span class="exp-vc-model opus">Opus</span>
    </a>
    <a href="...-v4.html" class="exp-vc-entry">
      <span class="exp-vc-v">v4</span>
      <span class="exp-vc-desc">Short description</span>
      <span class="exp-vc-model sonnet">Sonnet</span>
    </a>
    <!-- ... down to v1 -->
  </div>
</div>
```

- List versions in reverse chronological order (most recent first)
- Keep descriptions to a few words summarizing what that version changed
- The current version gets the `current` class
- Include `<span class="exp-vc-model {model-class}">{Model}</span>` on each entry to show which model produced that version

---

## Agent Trace Pattern

Every experiment sidebar includes an Agent Trace section that documents all agents spawned during creation. This provides full provenance — which agents ran, what model each used, what they were told to do, and in what order.

### When to Include

Always include the Agent Trace section in every experiment sidebar. Even for simple single-agent sessions, document the Lead agent entry. The trace is the record of how the experiment was built.

### Placement

Place the Agent Trace section **after the Prompt section and before the Versions section** in the sidebar.

### Agent Trace CSS Pattern

```css
/* ─── Agent Trace ─── */
.experiment-sidebar .exp-at-workflow {
  font-size: 10px;
  color: #888;
  font-family: 'SF Mono', 'Fira Code', monospace;
  padding: 6px 10px;
  background: rgba(255,255,255,0.03);
  border-radius: 4px;
  margin-bottom: 8px;
  letter-spacing: 0.02em;
}

.experiment-sidebar .exp-agent-trace {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.experiment-sidebar .exp-at-entry {
  background: rgba(255,255,255,0.04);
  border-radius: 6px;
  padding: 8px 10px;
  border-left: 3px solid #555;
}

.experiment-sidebar .exp-at-entry.opus { border-left-color: #e5a63b; }
.experiment-sidebar .exp-at-entry.sonnet { border-left-color: #8b9cf7; }
.experiment-sidebar .exp-at-entry.haiku { border-left-color: #5ec4d4; }

.experiment-sidebar .exp-at-header {
  display: flex;
  align-items: center;
  gap: 6px;
  margin-bottom: 4px;
}

.experiment-sidebar .exp-at-role {
  font-size: 10px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.04em;
  color: #ddd;
}

.experiment-sidebar .exp-at-model {
  font-size: 9px;
  font-weight: 600;
  padding: 1px 5px;
  border-radius: 3px;
  margin-left: auto;
}
.experiment-sidebar .exp-at-model.opus { background: rgba(229,166,59,0.15); color: #e5a63b; }
.experiment-sidebar .exp-at-model.sonnet { background: rgba(139,156,247,0.15); color: #8b9cf7; }
.experiment-sidebar .exp-at-model.haiku { background: rgba(94,196,212,0.15); color: #5ec4d4; }

.experiment-sidebar .exp-at-action {
  font-size: 11px;
  color: #bbb;
  line-height: 1.4;
}

.experiment-sidebar .exp-at-arrow {
  text-align: center;
  color: #555;
  font-size: 10px;
  line-height: 1;
}

/* ─── Agent Trace: Duration Badge ─── */
.experiment-sidebar .exp-at-duration {
  font-size: 9px;
  font-weight: 600;
  padding: 1px 5px;
  border-radius: 3px;
  background: rgba(255, 255, 255, 0.06);
  color: #999;
  font-variant-numeric: tabular-nums;
  font-family: 'SF Mono', 'Fira Code', monospace;
}

/* ─── Agent Trace: Hover Timestamps ─── */
.experiment-sidebar .exp-at-times {
  max-height: 0;
  overflow: hidden;
  opacity: 0;
  transition: max-height 0.25s ease, opacity 0.25s ease, margin 0.25s ease;
  margin-top: 0;
}

.experiment-sidebar .exp-at-entry:hover .exp-at-times {
  max-height: 40px;
  opacity: 1;
  margin-top: 4px;
}

.experiment-sidebar .exp-at-time-row {
  display: flex;
  gap: 12px;
  font-size: 9px;
  font-family: 'SF Mono', 'Fira Code', monospace;
  font-variant-numeric: tabular-nums;
  color: #666;
  padding: 3px 0;
}

.experiment-sidebar .exp-at-time-label {
  color: #555;
  min-width: 32px;
}

.experiment-sidebar .exp-at-time-value {
  color: #888;
}

/* ─── Agent Trace: Full Prompt Toggle ─── */
.experiment-sidebar input[id^="at-prompt-"] {
  position: absolute;
  opacity: 0;
  pointer-events: none;
}

.experiment-sidebar .exp-at-prompt-toggle {
  display: inline-block;
  font-size: 8px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: #8b9cf7;
  background: rgba(139, 156, 247, 0.1);
  border: 1px solid rgba(139, 156, 247, 0.2);
  border-radius: 3px;
  padding: 0px 5px;
  margin-top: 4px;
  cursor: pointer;
  transition: background 0.2s, color 0.2s;
  user-select: none;
}

.experiment-sidebar .exp-at-prompt-toggle:hover {
  background: rgba(139, 156, 247, 0.2);
  color: #fff;
}

.experiment-sidebar input[id^="at-prompt-"]:checked + .exp-at-prompt-toggle {
  background: rgba(139, 156, 247, 0.25);
  color: #fff;
  border-color: rgba(139, 156, 247, 0.4);
}

.experiment-sidebar .exp-at-prompt-panel {
  max-height: 0;
  overflow: hidden;
  opacity: 0;
  transition: max-height 0.35s ease, opacity 0.25s ease, padding 0.35s ease;
  padding: 0;
}

.experiment-sidebar input[id^="at-prompt-"]:checked + .exp-at-prompt-toggle + .exp-at-prompt-panel {
  max-height: 800px;
  opacity: 1;
  padding-top: 6px;
}

.experiment-sidebar .exp-at-prompt-panel-inner {
  background: rgba(139, 156, 247, 0.06);
  border: 1px solid rgba(139, 156, 247, 0.15);
  border-radius: 6px;
  padding: 10px 12px;
  font-size: 10px;
  line-height: 1.5;
  color: #bbb;
  font-style: italic;
  max-height: 300px;
  overflow-y: auto;
}

.experiment-sidebar .exp-at-prompt-panel-inner::-webkit-scrollbar {
  width: 4px;
}
.experiment-sidebar .exp-at-prompt-panel-inner::-webkit-scrollbar-track {
  background: transparent;
}
.experiment-sidebar .exp-at-prompt-panel-inner::-webkit-scrollbar-thumb {
  background: rgba(139, 156, 247, 0.3);
  border-radius: 2px;
}

.experiment-sidebar .exp-at-prompt-panel-label {
  font-size: 8px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  color: #8b9cf7;
  margin-bottom: 4px;
  font-style: normal;
}
```

### Agent Trace HTML Pattern

```html
<div class="exp-section">
  <h2>Agent Trace</h2>
  <div class="exp-at-workflow">Lead → Researcher → Builder → Sidebar</div>
  <div class="exp-agent-trace">
    <div class="exp-at-entry opus">
      <div class="exp-at-header">
        <span class="exp-at-role">Lead</span>
        <span class="exp-at-duration">45s</span>
        <span class="exp-at-model opus">Opus 4.6</span>
      </div>
      <div class="exp-at-action">Planned approach, spawned 3 agents</div>
      <div class="exp-at-times">
        <div class="exp-at-time-row">
          <span class="exp-at-time-label">Start</span>
          <span class="exp-at-time-value">2:10:00 pm</span>
          <span class="exp-at-time-label">End</span>
          <span class="exp-at-time-value">2:10:45 pm</span>
        </div>
      </div>
    </div>
    <div class="exp-at-arrow">↓</div>
    <div class="exp-at-entry sonnet">
      <div class="exp-at-header">
        <span class="exp-at-role">Researcher</span>
        <span class="exp-at-duration">1m 23s</span>
        <span class="exp-at-model sonnet">Sonnet 4.5</span>
      </div>
      <div class="exp-at-action">Explored codebase patterns, read 12 files</div>
      <div class="exp-at-times">
        <div class="exp-at-time-row">
          <span class="exp-at-time-label">Start</span>
          <span class="exp-at-time-value">2:10:46 pm</span>
          <span class="exp-at-time-label">End</span>
          <span class="exp-at-time-value">2:12:09 pm</span>
        </div>
      </div>
      <!-- Full prompt toggle: checkbox ID must be unique within the file -->
      <input type="checkbox" id="at-prompt-1">
      <label for="at-prompt-1" class="exp-at-prompt-toggle">Prompt ↓</label>
      <div class="exp-at-prompt-panel">
        <div class="exp-at-prompt-panel-inner">
          <div class="exp-at-prompt-panel-label">Full Prompt</div>
          Explore the project structure and identify existing sidebar patterns.
          Read the CLAUDE.md file for conventions. Check the latest 3 experiment
          files for reusable CSS and HTML patterns...
        </div>
      </div>
    </div>
    <div class="exp-at-arrow">↓</div>
    <div class="exp-at-entry opus">
      <div class="exp-at-header">
        <span class="exp-at-role">Builder</span>
        <span class="exp-at-duration">3m 17s</span>
        <span class="exp-at-model opus">Opus 4.6</span>
      </div>
      <div class="exp-at-action">Built experiment HTML/CSS (7100 lines)</div>
      <div class="exp-at-times">
        <div class="exp-at-time-row">
          <span class="exp-at-time-label">Start</span>
          <span class="exp-at-time-value">2:12:10 pm</span>
          <span class="exp-at-time-label">End</span>
          <span class="exp-at-time-value">2:15:27 pm</span>
        </div>
      </div>
      <input type="checkbox" id="at-prompt-2">
      <label for="at-prompt-2" class="exp-at-prompt-toggle">Prompt ↓</label>
      <div class="exp-at-prompt-panel">
        <div class="exp-at-prompt-panel-inner">
          <div class="exp-at-prompt-panel-label">Full Prompt</div>
          Create a new version of the Everything App experiment with a complete
          Calendar module. Use HEY.com-inspired weekly timeline with CSS Grid,
          department color coding, 22 sample events across 5 departments...
        </div>
      </div>
    </div>
    <div class="exp-at-arrow">↓</div>
    <div class="exp-at-entry sonnet">
      <div class="exp-at-header">
        <span class="exp-at-role">Sidebar</span>
        <span class="exp-at-duration">52s</span>
        <span class="exp-at-model sonnet">Sonnet 4.5</span>
      </div>
      <div class="exp-at-action">Documented metadata, prompts, and agent trace</div>
      <div class="exp-at-times">
        <div class="exp-at-time-row">
          <span class="exp-at-time-label">Start</span>
          <span class="exp-at-time-value">2:15:28 pm</span>
          <span class="exp-at-time-label">End</span>
          <span class="exp-at-time-value">2:16:20 pm</span>
        </div>
      </div>
    </div>
  </div>
</div>
```

### Agent Trace Rules

- **Every agent gets an entry** — including quick Explore agents, Plan agents, and validation passes. Full provenance means full provenance.
- **Show agents in execution order** — top to bottom = first to last
- **Use `↓` arrows** between sequential steps. For parallel agents, omit arrows and place entries side-by-side (they'll stack vertically but without the arrow separator).
- **The workflow summary** (`.exp-at-workflow`) shows the pipeline as a one-liner: `Lead → Researcher → Builder → Sidebar`. Use `→` for sequential steps and `[A | B]` for parallel steps (e.g., `Lead → [Builder×10] → Validator`).
- **Model values**: Use the full model identifier (e.g., "Opus 4.6", "Sonnet 4.5", "Haiku 4.5") in the badge text.

#### Timestamps

- **Duration badge is mandatory** on every agent trace entry. Place it in the `.exp-at-header` between `.exp-at-role` and `.exp-at-model`.
- **Duration format**: `Xm Ys` for minutes+seconds (e.g., `1m 42s`). Use `Xs` for sub-minute durations (e.g., `45s`). Use `Xm` for round minutes (e.g., `3m`). For long-running agents, use `Xh Ym` (e.g., `1h 12m`).
- **Start/end timestamps** go in the `.exp-at-times` block immediately after `.exp-at-action`. These are hover-revealed — they expand smoothly when the user hovers over the entry.
- **Timestamp format**: `h:mm:ss am/pm` (e.g., `9:14:03 pm`). Use 12-hour format with lowercase am/pm.
- **When timestamps are required**: Always for Builder and Researcher agents. Always for any agent that runs longer than 30 seconds. Optional for Lead and Sidebar agents where timing is less meaningful, but including them is preferred.
- **How to determine timing**: Use the session context — note wall-clock time when spawning an agent (start) and when receiving its result (end). Duration = end - start. For parallel agents, start times may be very close; end times will differ based on agent complexity.

#### Full Prompts

- **Full prompt toggle is mandatory** for every agent that receives a substantive prompt (Builder, Researcher, Validator). This replaces the old `.exp-at-prompt` truncated text.
- **The full prompt toggle is optional** for Lead and Sidebar agents where the context is self-evident from the action description.
- **Checkbox ID convention**: Use `at-prompt-{N}` where N is sequential within the experiment file, starting at 1. Example: `at-prompt-1`, `at-prompt-2`, `at-prompt-3`. Each ID must be unique within the file.
- **HTML ordering is critical**: The checkbox `<input>`, `<label>`, and `.exp-at-prompt-panel` must appear as adjacent siblings in exactly that order. The `:checked + label + panel` selector chain will break if anything is inserted between them.
- **Prompt content must be the full verbatim prompt** given to the agent — not truncated, not paraphrased. Use `<br><br>` between paragraphs for readability. This is the complete build instruction the agent received.
- **The `.exp-at-prompt-panel-inner` scrolls** at 300px max-height with a custom scrollbar. Even very long prompts (1000+ chars) are handled gracefully.

---

## Origination Prompt Pattern

The origination prompt pattern is used when a single master prompt spawns multiple autonomous experiment variants through parallel execution, rather than sequential iteration.

### When to Use

Use the origination pattern when:
- A research-driven prompt generates N parallel variants (typically 5–10)
- Each variant explores a different interpretation, persona, or UI approach
- All variants are created autonomously in the same session
- The variants form a cohesive research series, not iterations of each other

### File Structure

1. **Origin page**: `{family}.html` — An HTML page using the standard sidebar pattern that displays the research context, master prompt, and links to all variants
2. **Variant files**: `{family}-v1.html` through `{family}-v10.html` — Use the same `-v{N}` suffix as standard experiments. Each is a standalone experiment. All variants created in parallel, not sequentially.

### Origin Page

The origin page (`{family}.html`) uses the standard experiment sidebar but with an "Origin" badge instead of a version badge. The content area displays:
- **The exact verbatim origination prompt** — copy-pasted, not paraphrased. This is the most important element. Preserve the original text exactly as the user typed it, including any typos or informal phrasing. Use `<br><br>` between paragraphs.
- **All AskUserQuestion interactions** — If the AskUserQuestion tool was used during the session to refine the direction, include each interaction with: the question asked, all options presented, which option was selected (or "Other"), and the user's answer text. This preserves the full conversational context that shaped the final output. Use the AskUserQuestion visualization pattern (see below).
- Research domains and context
- A grid/list of all variants with descriptions
- Execution model notes (research agent → N parallel build agents → validation agent)

The prompt and AskUserQuestion interactions are the primary value of the origin page — they document *how* the human and agent worked together to shape the experiment series. Without them, the origin page is incomplete.

### Prompt Preservation — Non-Negotiable Rule

**Nothing is more important than documenting the prompts.** The entire purpose of this framework is to show how humans and agents work together to build experiments. Every origin page MUST include the verbatim origination prompt and every AskUserQuestion interaction. Every variant MUST include the exact build prompt given to its sub-agent. Summaries, paraphrases, and "descriptions of intent" are never acceptable substitutes for the actual prompt text. If a prompt is lost, the experiment's documentation is broken.

### Experiment Sidebar for Variants

Each variant experiment uses a modified sidebar with these differences from the standard pattern:

- **Variant badge** (green) instead of version badge (blue): `<span class="exp-variant">v1</span>`
- **"Fully Autonomous" badge** always present
- **Variant Focus section** explaining what makes THIS variant unique
- **Prompt section** — **MANDATORY.** Must contain the **exact, complete prompt** that was given to the sub-agent that built this variant. Not a summary, not a paraphrase — the full build prompt as it was sent. This is the single most important piece of documentation in the sidebar. Prepend with: *"This version was generated autonomously by a sub-agent. The prompt below is the exact prompt given to the build agent — it was crafted by the orchestrating agent based on the origination prompt and research phase."*
- **Origination Prompt section** linking to the origin page
- **Versions section** using the Sidebar Versions Card pattern for navigating between siblings

### Origination Variant Sidebar CSS

Add this CSS to every variant experiment file (in addition to the standard sidebar CSS):

```css
/* ─── Origination Variant Badge ─── */
.experiment-sidebar .exp-variant {
  display: inline-block;
  background: rgba(46, 204, 113, 0.15);
  color: #2ecc71;
  font-size: 11px;
  font-weight: 600;
  padding: 2px 8px;
  border-radius: 4px;
  margin-top: 4px;
}

.experiment-sidebar .exp-origination-link {
  font-size: 12px;
  line-height: 1.6;
  color: #ccc;
}

.experiment-sidebar .exp-origination-link a {
  color: #2ecc71;
  font-weight: 600;
}

/* Note: Variant navigation now uses the Sidebar Versions Card pattern (see above) */
```

### Origination Variant Sidebar HTML Pattern

```html
<aside class="experiment-sidebar">
  <div>
    <a href="experiments.html" class="exp-back">← All Experiments</a>
    <h1>{Experiment Family}</h1>
    <span class="exp-variant">v{N}</span>
    <span class="exp-model opus">Opus 4.6</span>
    <span class="exp-autonomous">Fully Autonomous</span>
  </div>

  <div class="exp-section">
    <h2>Purpose</h2>
    <p>{What this specific variant explores}. Part of a {N}-variant research series.</p>
  </div>

  <div class="exp-section">
    <h2>Variant Focus</h2>
    <p>{Specific UI approach, persona, or design direction for THIS variant}.</p>
  </div>

  <div class="exp-section">
    <h2>Prompt</h2>
    <div class="exp-prompt">
      This version was generated autonomously by a sub-agent. The prompt below is the exact prompt given to the build agent — it was crafted by the orchestrating agent based on the origination prompt and research phase.<br><br>
      {THE FULL, EXACT BUILD PROMPT THAT WAS GIVEN TO THE SUB-AGENT. Include variant name, tagline, core metaphor, layout description, key UI elements, color/mood direction, role toggle behavior — everything the agent received. Use <br><br> between paragraphs. Use <strong> for the variant name header.}
    </div>
  </div>

  <div class="exp-section">
    <h2>Origination Prompt</h2>
    <p class="exp-origination-link">
      This experiment is part of a research-driven series.
      <a href="{family}.html">View the origination prompt →</a>
    </p>
  </div>

  <div class="exp-section">
    <h2>Versions</h2>
    <div class="exp-versions-card">
      <a href="{family}.html" class="exp-vc-origin">Origination Prompt →</a>
      <a href="{family}-v3.html" class="exp-vc-entry">
        <span class="exp-vc-v">v3</span>
        <span class="exp-vc-desc">Variant concept name</span>
      </a>
      <a href="{family}-v2.html" class="exp-vc-entry current">
        <span class="exp-vc-v">v2</span>
        <span class="exp-vc-desc">Variant concept name</span>
      </a>
      <a href="{family}-v1.html" class="exp-vc-entry">
        <span class="exp-vc-v">v1</span>
        <span class="exp-vc-desc">Variant concept name</span>
      </a>
    </div>
  </div>
</aside>
```

### experiments.html — Origination Series Card

Add one card per origination series. Use the "Origination Series" badge (green) and list all variants in the changelog.

#### Origination CSS (add to experiments.html)

```css
/* ─── Origination Series Styling ─── */
.badge-origination {
  background: rgba(46, 204, 113, 0.12);
  color: #2ecc71;
}

.card-origination {
  border-color: rgba(46, 204, 113, 0.15);
}
```

#### Origination Card HTML Template

```html
<!-- {Experiment Family} (Origination Series) -->
<div class="card card-origination">
  <div class="card-accent"></div>
  <div class="card-body">
    <div class="card-meta">
      <h3>{Experiment Family}</h3>
      <span class="badge badge-updated">Latest</span>
      <span class="badge badge-origination">Origination Series</span>
      <span class="badge badge-versions">{N} variants</span>
    </div>
    <p class="desc">{Brief description of the research series.}</p>
    <div class="tags">
      <span class="tag">{tag1}</span>
      <span class="tag">research-series</span>
    </div>
    <div class="changelog">
      <!-- One entry per variant -->
      <a href="{family}-v1.html" class="changelog-entry latest">
        <span class="changelog-v">v1</span>
        <span class="changelog-time">{time}</span>
        <span class="changelog-desc autonomous">{Variant concept name}</span>
        <span class="changelog-model {model-class}">{Model}</span>
        <span class="changelog-detail">{1-2 sentence description}</span>
      </a>
      <!-- ... more variants ... -->
      <a href="{family}.html" class="view-latest">View origination prompt →</a>
    </div>
  </div>
</div>
```

#### Hero Stats Update

When adding an origination series, count it as 1 experiment. Count each variant as 1 version. Add a third stat for origination series count:

```html
<div class="stats">
  <span><span class="dot"></span> {X} experiments</span>
  <span><span class="dot"></span> {Y} versions</span>
  <span><span class="dot"></span> {Z} origination series</span>
</div>
```

### Multi-Origination Series

When a single origination series spawns multiple waves of variants (e.g., v1-v10 from prompt #1, then v11-v20 from prompt #2), use this pattern:

**Origin page structure**:
- Add `id="origination-1"` and `id="origination-2"` (etc.) anchors to each prompt heading
- Document each origination prompt in its own section with verbatim text
- Show separate variant grids for each wave
- Update execution model to show multiple origination phases

**experiments.html card**:
- Use inline origination dividers within the changelog to separate waves
- Place each divider BEFORE the first variant of its wave (chronologically after)
- Link dividers to `origin-file.html#origination-N`

#### Changelog Origination Divider Template

```html
<div class="changelog-origination-divider">
  <span class="cod-label">Origination #{N}</span>
  <a href="{family}.html#origination-{N}" class="cod-link">{Short description} →</a>
</div>
```

#### Changelog Origination Divider CSS

```css
.changelog-origination-divider {
  background: rgba(46, 204, 113, 0.08);
  border: 1px solid rgba(46, 204, 113, 0.2);
  border-radius: 6px;
  padding: 10px 14px;
  margin: 6px 0;
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: default;
}
.changelog-origination-divider .cod-label {
  font-size: 10px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: #2ecc71;
  flex-shrink: 0;
}
.changelog-origination-divider .cod-link {
  font-size: 11px;
  color: #2ecc71;
  text-decoration: none;
  font-weight: 600;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  min-width: 0;
}
.changelog-origination-divider .cod-link:hover {
  text-decoration: underline;
}
```

**Variant sidebars**:
- Link to the specific origination prompt anchor (e.g., `#origination-2`)
- Versions card shows ALL variants from all waves
- List origination prompt links at top of versions card (most recent first)
- Use group labels to separate waves visually:

```css
.experiment-sidebar .exp-vc-group-label {
  font-size: 9px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: #666;
  padding: 8px 10px 4px;
}
```

```html
<div class="exp-versions-card">
  <a href="{family}.html#origination-2" class="exp-vc-origin">Origination Prompt #2 →</a>
  <a href="{family}.html#origination-1" class="exp-vc-origin">Origination Prompt #1 →</a>
  <div class="exp-vc-group-label">Wave 2 — Enhanced</div>
  <!-- v20-v11 entries -->
  <div class="exp-vc-group-label">Wave 1 — Original</div>
  <!-- v10-v1 entries -->
</div>
```

**changelog.html**:
- One meta entry per origination prompt addition
- One file creation entry per variant created

---

## Team System

Experiments are built using a formalized team of specialized agents. Each role has defined responsibilities, a default model, and clear handoff points.

### Defined Roles

| Role | Responsibility | Default Model | When to Use |
|------|---------------|---------------|-------------|
| **Lead** | Orchestrates workflow, plans approach, delegates tasks, synthesizes results, updates `experiments.html` and `changelog.html` | Opus | Always — this is the main agent |
| **Researcher** | Explores codebase, gathers context, finds reusable patterns, reads files | Sonnet | When exploration is needed before building |
| **Builder** | Creates experiment HTML/CSS content. Focuses exclusively on the experiment UI. | Opus | Always for experiment creation |
| **Sidebar** | Documents sidebar metadata: purpose, prompt, agent trace, versions card, model badges. Operates on the completed experiment file. | Sonnet | Always — runs after Builder completes |
| **Validator** | Reviews output against CLAUDE.md standards, checks links, verifies CSS patterns, tests device preview | Sonnet | For complex experiments or origination series |

### Model Selection Rules

Choose models based on task complexity:
- **Opus**: Creative work, complex CSS, layout design, multi-module experiments, orchestration
- **Sonnet**: Codebase exploration, pattern matching, structured documentation, validation, sidebar generation
- **Haiku**: Quick searches, simple file reads, validation of small isolated changes

When spawning subagents via the Task tool, set the `model` parameter explicitly. The model used must be documented in the Agent Trace.

### Workflow Modes

**Simple experiment** (new version of existing):
```
Lead → Researcher (optional) → Builder → Sidebar
```

**Complex experiment** (new experiment or major version):
```
Lead → Researcher → Builder → Sidebar → Validator
```

**Origination series** (parallel variant generation):
```
Lead → Researcher → [Builder×N] (parallel) → [Sidebar×N] (parallel) → Validator
```

### Sidebar Agent Responsibilities

The Sidebar agent receives the completed experiment file and is responsible for:
1. Writing all sidebar HTML with required sections (purpose, prompt, agent trace, versions card)
2. Populating the Agent Trace from session context — every agent that ran, with role, model, action, duration, timestamps, and full prompt
3. **Recording timestamps** — Note the wall-clock time when each agent was spawned (start) and when its result was received (end). Compute duration as end - start. Format as `Xm Ys` (e.g., `1m 42s`).
4. **Generating prompt toggle IDs** — Assign sequential `at-prompt-{N}` IDs starting at 1 for each agent that receives a full prompt toggle. Ensure all IDs are unique within the file.
5. **Including the full verbatim prompt** in each agent's `.exp-at-prompt-panel-inner` — the complete text given to the agent, not truncated
6. Setting the correct model badge (`.exp-model`) and version badge (`.exp-version`)
7. Setting autonomy indicators if applicable (`.exp-autonomous`)
8. Preserving the verbatim user prompt in the `.exp-prompt` section
9. Building the Versions card with all existing versions and model indicators
10. Including all AskUserQuestion interactions in the prompt section

### Synthesis Documentation

When the Lead agent synthesizes work from multiple agents (e.g., merging Builder output with Sidebar output, or combining parallel variant builds), the Agent Trace should include a final Lead entry documenting the synthesis step:

```html
<div class="exp-at-entry opus">
  <div class="exp-at-header">
    <span class="exp-at-role">Lead</span>
    <span class="exp-at-model opus">Opus 4.6</span>
  </div>
  <div class="exp-at-action">Synthesized Builder + Sidebar output, updated experiments.html and changelog.html</div>
</div>
```
