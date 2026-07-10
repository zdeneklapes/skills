---
name: project-documentation
description: Use when creating, reviewing, or improving project documentation structure, README files, AGENTS.md files, MkDocs docs, application functionality requirements, or architecture docs. Also use before implementing features that add, change, or remove functionality requirements. Do not use for code-only work unless documentation must change.
---

# Project Documentation Skill

Use this skill when writing, reviewing, or improving project documentation. The goal is documentation that is current, concrete, easy to navigate, rendered by MkDocs when applicable, and useful for both humans and coding agents.

## Core Principle

Each document should have one clear job.

Do not duplicate the same information in many places. Put full details in one canonical document and link to it from other places.

Project documentation should cover both application requirements (what the application should do) and implementation details (how the code works).

## Requirements First

Any addition, change, or removal of an application functionality requirement must be reflected in `docs/docs/requirements/` before implementation. Then implement in the Recommended Implementation Order from `references/application-requirements.md`.

For all other documentation (README, `AGENTS.md`, architecture), update it when the user asks for documentation, the task explicitly requires documentation, or the change clearly makes existing documented behavior inaccurate. Do not write new documentation files for ordinary code changes.

Write architecture documentation only when the user or developer explicitly asks for it. Not otherwise.

## Load References

Read only the reference files that match the documentation task.

| Task | Reference |
| --- | --- |
| Product functionality, requirements indexes, product scope, glossary, status tables, acceptance criteria, roadmap, implementation order, or status-filter tables | `references/application-requirements.md` |
| Architecture docs, code flows, module ownership, diagrams, technical decisions, security, access, billing, sharing, user-generated content, privacy, retention, non-functional requirements, integrations, telemetry, migration, or possible issues | `references/architecture-and-cross-cutting.md` |

For broad project-documentation setup or review, read every relevant reference before editing.

## Documentation Placement

| File or folder | What belongs there |
| --- | --- |
| `README.md` | Human overview, setup, quickstart, basic commands, environment overview, and links. |
| `AGENTS.md` (project root) | All agent-facing guidance. See Project AGENTS.md below. |
| `docs/docs/requirements/` | Application functionality requirements and their `index.md`. |
| `docs/docs/architecture/` | Current technical design: how the code works now. Written only on explicit request. |
| `docs/docs/assets/` | MkDocs CSS and JavaScript for requirement status tables. |
| Skills | Repeatable agent workflows. |
| CI, hooks, scripts | Enforced checks. |

Do not create `docs/docs/agents/`, `docs/docs/adr/`, `docs/docs/operations/`, or `docs/docs/plans/` folders. Agent-facing content belongs in the root `AGENTS.md`. Record important technical decisions (why, alternatives, consequences) inside the relevant architecture page.

## Project AGENTS.md

Everything agent-related lives in the single project-root `AGENTS.md`. Keep it short: reference documentation folders, not every leaf file.

Required sections:

* Project: purpose and what must not be broken.
* Where Things Are: paths table for code, tests, and docs.
* Project Documentation: links to `README.md` and `docs/docs/` folders.
* Commands: verified commands, where to run them, and what they check.
* Testing And Verification: fast tests, full tests, lint, typecheck.
* Update Map: files that must be updated together, and generated files with their generation commands.
* Safety: secrets, private data, destructive commands, and risky operations.
* Done: definition of when work is complete.

Do not put long architecture text, full API docs, database schemas, old logs, generated repo dumps, duplicated README content, vague advice, unverified instructions, or secrets in `AGENTS.md`.

Add nested `AGENTS.md` files only when a folder or subsystem needs special local rules. They should contain only local paths, references to relevant docs, conventions, verification pointers, and danger zones — never repeat the root file.

## MkDocs

* Keep docs under the configured MkDocs docs directory, for example `docs/docs/`, with navigation organized in `mkdocs.yml`.
* Keep Markdown valid and renderable.
* Status-filter asset rules live in `references/application-requirements.md`.

## Review Checklist

Before finalizing documentation, verify:

* The Core Principle holds: one job per document, no duplicated information.
* Requirement changes are reflected in `docs/docs/requirements/` before implementation, with formats matching the loaded references.
* MkDocs navigation and assets are updated when pages or assets change.

## Writing Style

Be clear and direct.

* Use exact paths, exact commands, stable IDs, tables, diagrams, flowcharts, sequence diagrams, examples, or screenshots when they make the logic easier to understand.
* Prefer measurable requirements over vague adjectives.
* Use one canonical term from the glossary.
* Avoid too many concrete examples.
* Avoid outdated notes, old logs, and implementation details unless they affect behavior, security, data, or delivery risk.
