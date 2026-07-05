# Documentation Structure Reference

Read this when deciding where project documentation should live, what a project-specific `AGENTS.md` should reference, how README and agent docs should be shaped, or how MkDocs files should be organized.

## Project AGENTS.md

A project-level `AGENTS.md` should stay short and reference the main documentation folders or index documents. Reference documentation folders, not every leaf file.

It should answer:

* What is this project?
* Where is the code?
* Where are the docs?
* How should work be verified?
* What must not be touched?

Add nested `AGENTS.md` files only when a folder or subsystem needs special local rules.

Keep detailed commands and workflows in `docs/docs/agents/commands.md` or the project's equivalent command document. Keep detailed change rules in `docs/docs/agents/update-map.md`, `docs/docs/agents/review-checklist.md`, or a dedicated change-rules document. Keep long details in `docs/docs/`. Keep repeated workflows in skills. Keep mandatory checks in CI, hooks, tests, linters, or scripts.

Do not put commands, detailed change rules, long architecture text, full API docs, database schemas, old logs, generated repo dumps, duplicated README content, vague advice, unverified instructions, or secrets in `AGENTS.md`.

Nested `AGENTS.md` files should contain only local paths, references to relevant docs, conventions, verification pointers, and danger zones. Do not repeat the root file.

## Recommended Project AGENTS.md Shape

```md
# AGENTS.md

## Project

Short description of the project and what must not be broken.

## Where Things Are

| Area | Path | Notes |
| --- | --- | --- |
| Backend | `backend/` | API, models, services. |
| Frontend | `frontend/` | UI, routes, client code. |
| Tests | `tests/` | Test suites. |
| Docs | `docs/` | Project documentation. |

## Project Documentation

@README.md
@docs/docs/application-functionality.md
@docs/docs/requirements/
@docs/docs/architecture/
@docs/docs/agents/
@docs/docs/operations/
@docs/docs/adr/

## Rules

- Follow existing patterns before adding new abstractions.
- Keep changes small unless refactoring is requested.
- Do not change public contracts silently.
- Do not edit generated files manually.
- Do not add dependencies without a clear reason.

## Done

Work is done when code is changed, referenced docs were followed, verification steps were completed or explained, docs/config were updated if needed, and the final response says what changed.

## Safety

Never commit secrets, keys, tokens, dumps, credentials, or private data. Do not run destructive commands, force-push, reset branches, delete data, or touch production unless explicitly requested.
```

## README.md

Put here:

* Project overview.
* Features.
* Requirements.
* Local setup.
* Basic run and test commands.
* Environment overview.
* Links to deeper docs.

Do not put here:

* Detailed agent rules.
* Large requirement tables.
* Full architecture docs.
* Generated indexes.
* Long troubleshooting logs.
* Internal prompting rules.

## Agent Docs

Use `docs/docs/agents/` for agent-facing implementation references.

Good files:

```text
docs/docs/agents/project-index.md
docs/docs/agents/commands.md
docs/docs/agents/testing.md
docs/docs/agents/update-map.md
docs/docs/agents/review-checklist.md
docs/docs/agents/security.md
docs/docs/agents/generated-files.md
```

Simple purposes:

* `project-index.md`: where important code lives and which tests cover it.
* `commands.md`: verified commands, where to run them, and what they check.
* `testing.md`: how to run fast tests, full tests, lint, typecheck, and focused tests.
* `update-map.md`: what files must be updated together.
* `review-checklist.md`: final checks before saying the task is done.
* `security.md`: secrets, private data, destructive commands, and risky operations.
* `generated-files.md`: generated files, source files, and generation commands.

## Plans

Write implementation plans into the project documentation plans folder.

Prefer:

```text
docs/docs/plans/
```

Create the directory if it does not exist.

## MkDocs Rules

All project docs should be valid Markdown rendered through MkDocs when the project uses MkDocs.

Rules:

* Keep docs under the configured MkDocs docs directory, for example `docs/docs/`.
* Keep navigation organized in `mkdocs.yml`.
* Use MkDocs assets for shared behavior, not inline scripts in every page.
* Put shared CSS and JavaScript in `docs/docs/assets/`.
* Register assets in `mkdocs.yml`.
* Keep Markdown valid and renderable.
* Use status-filter assets only for requirement/status tables.
* Keep requirement tables consistent so filtering works: include a `Status` column and use the same status names everywhere.

MkDocs paths in `extra_css` and `extra_javascript` are relative to the configured docs directory.

Register assets in `mkdocs.yml`, for example:

```yaml
extra_css:
  - assets/status-filter.css

extra_javascript:
  - assets/status-filter.js
```

## Update Docs When

Update docs when requirements change, commands change, folders move, APIs change, generated files change, architecture changes, deployment changes, test strategy changes, or the same agent mistake happens repeatedly.
