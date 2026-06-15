# The UI Lab

**A provenance-first framework for AI-built interface experiments.**

The UI Lab is a system for building, versioning, and exhibiting UI experiments made with AI. Each experiment is a single, self-contained HTML file — **pure HTML/CSS, no build step, no JavaScript, zero dependencies beyond Google Fonts** — that opens directly in a browser. What makes it different is that every file carries its own full provenance.

## The provenance model: prompt + model + effort = artifact

Every experiment records *how it was made*:

- **Prompt** — the exact, verbatim prompt that produced it (never paraphrased).
- **Model** — which Claude model ran (Opus / Sonnet / Haiku).
- **Effort** — the reasoning effort the agent ran at (high / medium / low).
- **Harness** — the agent harness (Claude Code).
- **Autonomy** — collaborative (a human and agent trading prompts) or fully autonomous (an agent given a direction and left to iterate).
- **Agent trace** — every agent that touched the file: role, model, duration, timestamps, and the full prompt it received.
- **Version lineage** — how the experiment evolved across `-v1`, `-v2`, …

The provenance lives in a left sidebar inside every experiment file, and in an immutable, append-only changelog. Anyone who reads the log later sees not just *what* changed and *why*, but *under exactly what conditions*.

## The three pages

- **`index.html`** — the landing page / pitch.
- **`experiments.html`** — the gallery. The **source of truth** for the experiment index.
- **`changelog.html`** — the immutable, append-only build log. Linked from the gallery footer.

Plus the experiment files themselves (`{name}-v1.html`, `{name}-v2.html`, …) and origination origin pages (`{family}.html`).

## Fork and run your own lab

1. **Fork** `github.com/ckluis/uiExplorer` (or use it as a template) and clone your copy.
2. Open **Claude Code** in the project directory. The `/uiExperiment` slash commands ship in `.claude/commands/uiExperiment/` and are available immediately.
3. Run a command — e.g. `/uiExperiment:new command-palette "explore a keyboard-first launcher"`. The agent scaffolds the file, builds the UI, documents the sidebar, and wires up `experiments.html` and `changelog.html` for you.
4. Open `experiments.html` in a browser to see your gallery, or push and view it on GitHub Pages.

Everything is governed by **`CLAUDE.md`** — the full rulebook the commands follow (file structure, versioning, gallery/changelog bookkeeping, sidebar patterns, the team system, origination series). Read it to understand the conventions; the commands automate them.

## The `/uiExperiment` commands

| Command | What it does |
|---------|--------------|
| `/uiExperiment:new [name] [purpose]` | Scaffold a new experiment `{name}-v1.html`: build content, document the sidebar, update `experiments.html`, log to `changelog.html`. |
| `/uiExperiment:version [name] [change]` | Create the next version `{name}-v{N+1}.html`: copy the latest, apply changes, backfill the Versions card in **all** prior versions, update `experiments.html`, log the creation. |
| `/uiExperiment:origination [family] [N] [direction]` | Run an origination series: an origin page + `N` parallel autonomous variants, registered as one origination card. |
| `/uiExperiment:changelog ["prompt"] [files]` | Append a `gc-meta` changelog entry for a manual/structural change, preserving the verbatim prompt + provenance chips. |
| `/uiExperiment:validate [name?]` | Audit the project against `CLAUDE.md` (one Latest badge, hero counts, sidebar sections, link integrity, freeze rule, changelog order) and offer to fix. |
| `/uiExperiment:help` | Print the command list, team roles, and workflow modes. |

### The team of roles

The commands delegate to a small team of Claude subagents, each recorded in the agent trace:

| Role | Responsibility | Default model |
|------|----------------|---------------|
| **Lead** | Orchestrates, plans, updates the gallery + changelog | Opus |
| **Researcher** | Explores the codebase, finds reusable patterns | Sonnet |
| **Builder** | Creates the experiment HTML/CSS | Opus |
| **Sidebar** | Documents purpose, prompt, agent trace, versions | Sonnet |
| **Validator** | Reviews output against the rules | Sonnet |

## Enabling GitHub Pages

The site is fully static, so it deploys to GitHub Pages with no build. Two ways:

### Option A — Deploy from a branch (simplest)
In your repo, go to **Settings → Pages → Build and deployment → Source → "Deploy from a branch"**, choose the **`main`** branch and the **`/ (root)`** folder, and save. Your lab goes live at `https://{your-username}.github.io/{repo}/`. The included `.nojekyll` file ensures Pages serves the files as-is (no Jekyll processing).

### Option B — GitHub Actions (included)
This repo ships `.github/workflows/deploy-pages.yml`, a workflow that deploys the repo root to GitHub Pages on every push to `main`. To use it: **Settings → Pages → Source → "GitHub Actions"**. The workflow uses the official `actions/configure-pages`, `actions/upload-pages-artifact`, and `actions/deploy-pages` actions with the correct `pages: write` / `id-token: write` permissions.

## Why pure HTML/CSS

No build step means every experiment works forever, opens instantly, and can be read as plain source. No JavaScript means the provenance can't drift from the artifact — what you see is what was made. The whole lab is just files you can fork, host anywhere, and read with your eyes.

---

🤖 The UI Lab framework is driven by [Claude Code](https://claude.com/claude-code).
