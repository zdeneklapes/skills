# Agent Skills

A small collection of agent skills installable with the `skills` CLI and discoverable by skills.sh after installation.

## Skills

| Skill | Description |
|---|---|
| [code-comments-and-documentation](skills/code-comments-and-documentation) | Write self-explaining code with comments, docstrings, and API docs that capture intent, invariants, and contracts. |
| [comments-with-references](skills/comments-with-references) | Write focused inline comments that explain non-obvious decisions and cite authoritative sources. |
| [project-documentation](skills/project-documentation) | Write and review README, AGENTS, requirements, architecture, ADR, and runbook documentation. |
| [writing-plans](skills/writing-plans) | Create detailed, testable implementation plans with explicit edge-case verification. |
| [shell-skill](skills/shell-skill) | Write and review explicit env-driven shell scripts, cron scripts, and helper wrappers. |
| [simplify-code](skills/simplify-code) | Refactor recently changed or targeted code for clarity while preserving behavior. |
| [python-development](skills/python-development) | Follow Python coding, testing, model-structure, and code-quality conventions. |
| [research-paper-writing](skills/research-paper-writing) | Draft, review, rewrite, and polish peer-ready research papers. |
| [test-driven-development](skills/test-driven-development) | Apply failing-test-first to material production behavior without testing every helper or implementation detail. |
| [vertical-slice-architecture](skills/vertical-slice-architecture) | Design, review, refactor, or implement backend, full-stack, and machine learning systems using Vertical Slice Architecture. |

## Installation

Install all skills from this repository:

```bash
npx skills add zdeneklapes/skills
```

Install one skill by name:

```bash
npx skills add zdeneklapes/skills --skill code-comments-and-documentation
npx skills add zdeneklapes/skills --skill comments-with-references
npx skills add zdeneklapes/skills --skill project-documentation
npx skills add zdeneklapes/skills --skill writing-plans
npx skills add zdeneklapes/skills --skill vertical-slice-architecture
npx skills add zdeneklapes/skills --skill simplify-code
npx skills add zdeneklapes/skills --skill shell-skill
npx skills add zdeneklapes/skills --skill python-development
npx skills add zdeneklapes/skills --skill research-paper-writing
npx skills add zdeneklapes/skills --skill test-driven-development
```

## Local Validation

List skills from a local checkout:

```bash
npx skills add . --list
```

Install one skill locally for Codex:

```bash
npx skills add . --skill code-comments-and-documentation -a codex -g
npx skills add . --skill comments-with-references -a codex -g
npx skills add . --skill project-documentation -a codex -g
npx skills add . --skill writing-plans -a codex -g
npx skills add . --skill vertical-slice-architecture -a codex -g
npx skills add . --skill simplify-code -a codex -g
npx skills add . --skill shell-skill -a codex -g
npx skills add . --skill python-development -a codex -g
npx skills add . --skill research-paper-writing -a codex -g
npx skills add . --skill test-driven-development -a codex -g
```

## Structure

Skills live under `skills/<skill-name>/SKILL.md`. Optional product metadata belongs in `skills/<skill-name>/agents/`. Longer supporting material belongs in `references/` and is linked from `SKILL.md` so agents can load it only when needed.
