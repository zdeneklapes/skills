---
name: project-documentation
description: Use when creating, reviewing, or improving project documentation structure, README files, AGENTS.md files, MkDocs docs, requirements indexes, architecture docs, ADRs, operations docs, or implementation plans. Do not use for code-only work unless documentation must change.
---

# Project Documentation Skill

Use this skill when writing, reviewing, or improving project documentation.

The goal is documentation that is current, concrete, easy to navigate, rendered by MkDocs when applicable, and useful for both humans and coding agents.

## Core Principle

Each document should have one clear job.

Do not duplicate the same information in many places. Put full details in one canonical document and link to it from other places.

After changing application logic, workflow behavior, public functionality, or functional requirements, update the relevant project documentation when the user asks for documentation, the task explicitly requires documentation, or the change clearly makes existing documented behavior inaccurate.

Do not write new documentation files for ordinary code changes unless the user asks for documentation, the task explicitly requires docs, or existing docs would become wrong without the update.

Project documentation should cover both:

1. Application requirements: what the application should do.
2. Implementation details: how the code works.

Use diagrams, flowcharts, state diagrams, sequence diagrams, tables, examples, or screenshots when they make the logic easier to understand.

## Load References

Read only the reference files that match the documentation task.

| Task | Reference |
| --- | --- |
| README, `AGENTS.md`, nested `AGENTS.md`, MkDocs layout, agent docs, plans, or docs placement | `references/documentation-structure.md` |
| Product functionality, requirements indexes, status tables, acceptance criteria, roadmap, or implementation order | `references/application-requirements.md` |
| Architecture docs, code flows, module ownership, package choices, diagrams, or ADRs | `references/architecture-and-decisions.md` |
| Authentication, authorization, data ownership, paid access, sharing, user-generated content, or privacy | `references/security-and-access.md` |
| Non-functional requirements, integrations, telemetry, audit logs, migration, backward compatibility, or possible issues | `references/operations-quality-and-risk.md` |

For broad project-documentation setup or review, read every relevant reference before editing.

## Documentation Placement

Use this map when deciding where documentation belongs.

| File or folder | What belongs there |
| --- | --- |
| `README.md` | Human overview, setup, quickstart, basic commands, environment overview, and links. |
| `AGENTS.md` | Short agent navigation guide: project purpose, paths, docs, verification, safety, and done definition. |
| Nested `AGENTS.md` | Special local rules for one folder or subsystem. |
| `docs/docs/application-functionality.md` | Product requirements index: what the application should do. |
| `docs/docs/requirements/` | Detailed functionality requirements before implementation. |
| `docs/docs/architecture/` | Current technical design: how the code works now. |
| `docs/docs/adr/` | Decision log: important technical decisions, alternatives, and consequences. |
| `docs/docs/agents/` | Agent-facing implementation references. |
| `docs/docs/operations/` | Deployment, rollback, monitoring, and runbooks. |
| `docs/docs/plans/` | Implementation plans and staged execution notes. |
| `docs/docs/assets/` | MkDocs CSS and JavaScript assets. |
| Skills | Repeatable agent workflows. |
| CI, hooks, scripts | Enforced checks. |

Use `architecture/` for how the system works now. Use `adr/` for why an important technical decision was made.

## Review Checklist

Before finalizing documentation, verify:

* The document has one clear job.
* Information is not duplicated across README, AGENTS, requirements, architecture, ADR, operations, and agent docs.
* README is human-facing.
* `AGENTS.md` is short and links to deeper docs.
* Requirements use stable IDs, statuses, targets, and testable acceptance criteria.
* Architecture docs describe how the code works now.
* ADRs explain why important technical decisions were made.
* MkDocs navigation and assets are updated when pages or assets change.
* Security, permissions, data access, integrations, migrations, and possible issues are documented when they affect the feature.
* The document stays concrete, current, and easy to scan.

## Writing Style

Be clear and direct.

Rules:

* Use exact paths, exact commands, stable IDs, tables, and diagrams when useful.
* Prefer measurable requirements over vague adjectives.
* Use one canonical term from the glossary.
* Avoid too many concrete examples.
* Avoid outdated notes, old logs, and implementation details unless they affect behavior, security, data, or delivery risk.
* Keep the skill lean by including only rules that change product behavior, implementation decisions, testing, security, or operational readiness.
