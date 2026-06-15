---
description: Overview of the /uiExperiment command suite — the commands, the team roles, the workflow modes, and where to read the rules.
---

# /uiExperiment:help — The UI Lab command suite

Print the following overview to the user. This is a self-documenting framework for building pure-HTML/CSS UI experiments where every artifact records its provenance: **prompt + model + effort = artifact**. The full rulebook is `CLAUDE.md` in the project root.

## Commands

| Command | What it does |
|---------|--------------|
| `/uiExperiment:new [name] [purpose]` | Scaffold a brand-new experiment `{name}-v1.html` from the template, build the content (Builder), document the sidebar (Sidebar), then update `experiments.html` (new-experiment rules) and append a `changelog.html` creation entry. |
| `/uiExperiment:version [name] [change]` | Create the next version `{name}-v{N+1}.html`: copy the latest file, apply changes, refresh its sidebar, **backfill the Versions card in ALL prior version files** (the only permitted frozen-file edit), update `experiments.html` (new-version rules), append a changelog creation entry. |
| `/uiExperiment:origination [family] [N] [direction]` | Run an origination series: build the origin page `{family}.html` with the verbatim master prompt, fan out `N` parallel Builder agents to produce autonomous variants `{family}-v1..vN.html`, add the origination card + dividers to `experiments.html`, bump all three hero stats, append changelog entries (1 meta + N+1 creations). |
| `/uiExperiment:changelog ["prompt"] [files]` | Append a `gc-meta` entry to `changelog.html` for any manual/structural change, with the VERBATIM prompt in `.gc-prompt-inner` plus provenance chips. |
| `/uiExperiment:validate [name?]` | Audit the project against `CLAUDE.md`: one Latest badge, hero counts match files, required sidebar sections present, links resolve, freeze rule respected, changelog newest-on-top. Reports a pass/fail checklist and offers to fix. |
| `/uiExperiment:help` | This overview. |

## Team roles (see CLAUDE.md → Team System)

| Role | Responsibility | Default model |
|------|----------------|---------------|
| **Lead** | Orchestrates, plans, delegates, updates `experiments.html` + `changelog.html` | Opus |
| **Researcher** | Explores codebase, finds reusable patterns | Sonnet |
| **Builder** | Creates the experiment HTML/CSS content | Opus |
| **Sidebar** | Documents purpose, prompt, agent trace, versions card, model badges | Sonnet |
| **Validator** | Reviews output against the rules, checks links/CSS/preview | Sonnet |

## Workflow modes

- **Simple** (new version): `Lead → Researcher (optional) → Builder → Sidebar`
- **Complex** (new experiment / major version): `Lead → Researcher → Builder → Sidebar → Validator`
- **Origination series** (parallel variants): `Lead → Researcher → [Builder×N] → [Sidebar×N] → Validator`

## Provenance model

Every command that writes a changelog entry or sidebar metadata records: **Model**, **Effort** (the `/effort` reasoning setting — high / medium / low; commands ASK if it can't be inferred), **Harness** (Claude Code), and **Autonomy** (collaborative vs autonomous). The changelog uses the `gc-model` and `gc-effort` chip classes; sidebars use the `exp-model` badge and the `exp-autonomous` indicator.

## Where to read the rules

- `CLAUDE.md` → **File Structure**, **Naming Convention** — how files are named and what the base `{name}.html` is reserved for.
- `CLAUDE.md` → **Updating experiments.html** (Adding a New Experiment / Adding a New Version) — hero stats, Latest badge, cards, changelog entries.
- `CLAUDE.md` → **Updating changelog.html** — file-creation vs meta entries, newest-on-top, prompt preservation.
- `CLAUDE.md` → **Standard Experiment Sidebar**, **Agent Trace Pattern**, **Sidebar Versions Card**, **AskUserQuestion**, **Device Preview Switcher** — the sidebar building blocks.
- `CLAUDE.md` → **Origination Prompt Pattern** — origin page + variant sidebars + the prompt-preservation rule.
- `CLAUDE.md` → **Team System** — roles, model selection, workflow modes.
- `README.md` — project overview, fork-and-run guide, GitHub Pages setup.
