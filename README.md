# Agent Skills

A small collection of agent skills installable with the `skills` CLI and discoverable by skills.sh after installation.

## Skills

| Skill | Description |
|---|---|
| [vertical-slice-architecture](skills/vertical-slice-architecture) | Design, review, refactor, or implement backend, full-stack, and machine learning systems using Vertical Slice Architecture. |

## Installation

Install all skills from this repository:

```bash
npx skills add <owner>/<repo>
```

Install only the Vertical Slice Architecture skill:

```bash
npx skills add <owner>/<repo> --skill vertical-slice-architecture
```

Replace `<owner>/<repo>` with the GitHub repository path after publishing.

## Local Validation

List skills from a local checkout:

```bash
npx skills add . --list
```

Install locally for Codex:

```bash
npx skills add . --skill vertical-slice-architecture -a codex -g
```

## Structure

Skills live under `skills/<skill-name>/SKILL.md`. Longer supporting material belongs in `references/` and is linked from `SKILL.md` so agents can load it only when needed.
