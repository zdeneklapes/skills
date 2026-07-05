---
name: simplify-code
description: Use when simplifying or refactoring existing code for readability, reducing complexity, removing duplication, deslopifying generated code, or preparing changed code for review while preserving behavior. Do not use for new features, behavior changes, or architecture redesign.
---

# Simplify Code

## Overview

Refine code so it is easier to read, maintain, test, and review without changing what it does. Prefer project conventions and explicit, boring code over clever rewrites or line-count reduction.

## Workflow

1. Establish scope.
   - Default to recently changed code: inspect `git status`, `git diff`, and relevant untracked files.
   - If the user names files, symbols, a PR, or a broader area, use that scope instead.
   - Avoid unrelated cleanup. Do not rewrite untouched areas just because they could be nicer.

2. Read before editing.
   - Identify the behavior the code must preserve, including tests, public APIs, edge cases, data formats, errors, and side effects.
   - Check project instructions and nearby code for conventions before applying generic preferences.
   - If behavior is unclear, inspect callers, tests, docs, or usage examples until the intended contract is concrete enough to preserve.

3. Simplify conservatively.
   Apply changes that reduce real complexity:
   - Flatten unnecessary nesting with guard clauses or clearer branching.
   - Replace nested ternaries and dense expressions with readable conditionals.
   - Remove dead code, unused variables, redundant wrappers, and duplicate logic.
   - Inline needless abstractions, but keep abstractions that express a meaningful domain concept or isolate complexity.
   - Improve names when the current names hide intent.
   - Split functions or components that mix distinct responsibilities.
   - Consolidate related logic when scattering makes the code harder to follow.
   - Remove comments that merely restate code; keep comments that explain non-obvious constraints, tradeoffs, or domain rules.
   - Prefer standard library and existing local helpers over bespoke code.

4. Preserve balance.
   Do not:
   - Change functionality, output shape, persistence, network behavior, timing-sensitive behavior, or public contracts unless explicitly asked.
   - Optimize for fewer lines when the result is harder to debug.
   - Introduce clever one-liners, implicit control flow, or broad abstractions to make code look polished.
   - Replace stable project patterns with personal style preferences.
   - Remove validation, error handling, accessibility, security checks, logging, or telemetry without proving they are unnecessary.
   - Mix simplification with feature work unless the user explicitly asks for both.

5. Verify.
   - Run the smallest relevant formatter, linter, typecheck, and tests that can validate the edited surface.
   - If no targeted command is obvious, inspect package or project scripts and choose a reasonable check.
   - If verification cannot run, state why and name the residual risk.

6. Report the result.
   - Summarize what was simplified and why.
   - Mention behavior-preservation evidence: tests run, typechecks, manual checks, or unchanged test expectations.
   - Call out any intentionally skipped simplifications where readability, risk, or project conventions argued against the change.

## Heuristics

Treat simplification as a refactor, not a redesign. A good result usually makes the existing intent more obvious with fewer moving parts. It does not surprise callers, invent new architecture, or erase useful structure.

Prefer these outcomes:

- Clear names over explanatory comments.
- Straight-line control flow over deeply nested blocks.
- Small cohesive units over large mixed-purpose units.
- Existing project idioms over generic textbook patterns.
- Targeted edits over broad churn.
- Verified behavior over assumed equivalence.

Be especially careful around:

- Authentication, authorization, payments, migrations, concurrency, caching, date/time logic, financial calculations, and security-sensitive validation.
- Generated files or vendored code.
- Public APIs and serialized data contracts.
- Tests that appear redundant but document edge cases.

## Example Prompts

- "Use $simplify-code on the files I just changed."
- "Simplify this component without changing behavior."
- "Run a /simplify-style pass before review."
- "Deslopify the generated code and keep the tests passing."
- "Reduce unnecessary complexity in this module."
