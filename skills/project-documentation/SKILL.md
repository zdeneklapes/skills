---
name: project-documentation
description: Use when creating, reviewing, or improving project documentation such as README files, AGENTS.md files, requirements, product specifications, architecture docs, ADRs, runbooks, or MkDocs sites; also use when code changes make those documents inaccurate. Do not use for code-only work with no documentation impact.
---

# Project Documentation

Create documentation that is accurate, useful, easy to navigate, and proportionate to the project. Write for both humans and coding agents when both audiences use the repository.

## Core Principles

Give each document a clear purpose. Keep detailed information in one canonical place and link to it elsewhere instead of maintaining competing copies.

Describe both what the product does and, where useful, how the implementation works. Keep verified current behavior distinct from plans, proposals, and assumptions.

## Adapt to the Project

Before proposing a structure, inspect the repository's existing documentation, configuration, navigation, and scoped `AGENTS.md` files.

Prefer the project's established layout and vocabulary unless the user asks for a reorganization or the current structure causes concrete problems. The paths, sections, tables, and status labels in this skill are starting points, not a universal schema.

Update documentation when:

* The user requests it or includes it in the deliverable.
* A change would make existing documentation inaccurate.
* A public contract, operational procedure, important constraint, or significant decision needs a durable record.

Do not create a new documentation system for an ordinary code change. If the project already maintains requirements or product specifications, update affected requirements before or alongside implementation.

## Load References

Read only the references relevant to the task.

| Task | Reference |
| --- | --- |
| Product scope, functionality, requirements, acceptance criteria, status tracking, roadmaps, or implementation dependencies | `references/application-requirements.md` |
| Architecture, decisions, code flows, security, access, billing, privacy, integrations, telemetry, migration, operations, quality, or risk | `references/architecture-and-cross-cutting.md` |

For a broad documentation setup or audit, read both references. Apply only the sections relevant to the project and request.

## Common Document Roles

| Document | Typical purpose |
| --- | --- |
| `README.md` | Project overview, setup, quickstart, common commands, and links to deeper documentation. |
| `AGENTS.md` | Actionable repository or subsystem guidance for coding agents. |
| Requirements or product specifications | Observable product behavior, scope, constraints, and acceptance criteria. |
| Architecture docs and ADRs | Current design, important flows, boundaries, decisions, and trade-offs. |
| Runbooks and operations docs | Procedures for deployment, recovery, maintenance, and incident response. |
| Plans | Temporary or durable implementation sequencing when the project uses plans. |
| CI, hooks, and scripts | Checks that are better enforced than described as optional prose. |

These roles do not require specific filenames or folders. Extend an existing canonical document when that is clearer than adding another one.

## AGENTS.md Guidance

Use a root `AGENTS.md` for repository-wide guidance and nested files when a subsystem needs local rules. Keep nested guidance scoped and avoid repeating the root file.

Useful topics include project purpose, important paths, verified commands, testing, files that change together, generated artifacts, safety constraints, and completion checks. Include only topics that help agents work correctly in that repository.

Keep long architecture explanations, complete API references, generated repository dumps, historical logs, duplicated README content, and secrets out of `AGENTS.md`. Link to their canonical locations when needed.

## Writing Guidance

* Verify claims against current code, configuration, or authoritative project sources.
* Use exact paths and commands when they are known and verified.
* Prefer measurable requirements and observable acceptance criteria.
* Use stable identifiers, tables, diagrams, examples, or screenshots when they improve maintenance or understanding, not by default.
* Preserve established domain terminology and explain important distinctions.
* Include implementation detail when it affects behavior, security, data, compatibility, operations, or delivery risk.

## Integrity and Safety

* Do not present guesses or planned behavior as current behavior.
* Do not mark work as supported or complete without evidence.
* Do not include secrets, credentials, private data, or unsafe operational instructions.
* For sensitive features, document relevant authorization, ownership, privacy, and failure behavior rather than treating frontend behavior as a security boundary.

## Review Checklist

Before finalizing documentation, check that:

* The result follows the repository's actual structure and conventions.
* Each important fact has one canonical home and links are current.
* Current, planned, unknown, and out-of-scope behavior remain distinguishable.
* Commands and implementation claims are verified or clearly qualified.
* Relevant navigation, indexes, and documentation assets are updated.
