# Agent Skills

A small collection of agent skills installable with the `skills` CLI and discoverable by skills.sh after installation.

## Skills

| Skill | Description |
|---|---|
| [shell-skill](skills/shell-skill) | Write and review explicit env-driven shell scripts, cron scripts, and helper wrappers. |
| [simplify-code](skills/simplify-code) | Refactor recently changed or targeted code for clarity while preserving behavior. |
| [python-structure-and-models](skills/python-structure-and-models) | Choose Pydantic models or dataclasses deliberately for Python structures and contracts. |
| [vertical-slice-architecture](skills/vertical-slice-architecture) | Design, review, refactor, or implement backend, full-stack, and machine learning systems using Vertical Slice Architecture. |

## Installation

Install all skills from this repository:

```bash
npx skills add zdeneklapes/skills
```

Install one skill by name:

```bash
npx skills add zdeneklapes/skills --skill vertical-slice-architecture
npx skills add zdeneklapes/skills --skill simplify-code
npx skills add zdeneklapes/skills --skill shell-skill
npx skills add zdeneklapes/skills --skill python-structure-and-models
```

## Local Validation

List skills from a local checkout:

```bash
npx skills add . --list
```

Install one skill locally for Codex:

```bash
npx skills add . --skill vertical-slice-architecture -a codex -g
npx skills add . --skill simplify-code -a codex -g
npx skills add . --skill shell-skill -a codex -g
npx skills add . --skill python-structure-and-models -a codex -g
```

## Structure

Skills live under `skills/<skill-name>/SKILL.md`. Optional product metadata belongs in `skills/<skill-name>/agents/`. Longer supporting material belongs in `references/` and is linked from `SKILL.md` so agents can load it only when needed.
