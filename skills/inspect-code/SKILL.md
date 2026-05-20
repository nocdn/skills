---
name: inspect-code
description: thoroughly inspect a codebase for refactor opportunities. use when the user asks to inspect code quality, find technical debt, reduce duplication, improve maintainability, identify dead code, centralize repeated logic, or suggest refactors without implementing them.
---

# Inspect Code

Inspect the codebase VERY thoroughly and produce a refactor-discovery report (It doesn't have to be very official like a report, just a list, with details, of things to refactor, including their impact, technical details, etc).

This skill is for discovery only. Do not edit files, move files, rename symbols, delete code, change APIs, or implement fixes unless the user explicitly asks for implementation later.

## Goal

Understand the code first, then identify refactor opportunities.

Look for ways to make the code smaller, clearer, less repetitive, less coupled, easier to test, easier to extend, easier to delete from, and easier for a future developer or AI agent to modify safely.

## Inspection approach

Do not jump to conclusions after reading one file.

Inspect the relevant files, nearby call sites, shared helpers, constants, types, schemas, services, hooks, utilities, tests, configuration, and project instructions. Build a clear mental model of how the code works before recommending changes.

When relevant, check files such as:

- AGENTS.md
- CLAUDE.md
- .github/copilot-instructions.md
- .cursorrules
- .cursor/rules
- README.md
- CONTRIBUTING.md
- architecture notes
- package scripts
- lint, test, and typecheck configuration

Follow the project’s existing conventions unless there is a clear reason not to.

If the repository is too large to inspect completely, inspect the most relevant areas first and clearly state what you inspected and what you did not inspect and why.

## What to look for

Use these as thinking lenses, not as an exhaustive checklist:

- duplicated logic
- repeated constants, strings, rules, types, schemas, or data shapes
- code that should probably be centralized
- oversized files, classes, functions, components, services, or modules
- long functions doing too many things
- unclear ownership of domain logic
- scattered business rules
- magic numbers or magic strings
- important concepts represented only as loose strings, numbers, booleans, or untyped objects
- weak abstraction boundaries
- UI code knowing too much about persistence, APIs, permissions, infrastructure, or backend details
- backend/controller code doing too many jobs at once
- repeated API, database, storage, validation, mapping, or error-handling patterns
- deeply nested conditionals
- boolean flags controlling too much behavior
- confusing or misleading names
- long parameter lists
- over-generalized abstractions
- premature abstractions
- utility files that have become junk drawers
- dead, unused, obsolete, unreachable, or commented-out code
- stale TODOs or misleading comments
- inconsistent error handling
- inconsistent validation
- inconsistent data mapping
- duplicated test setup
- weak or missing tests around risky logic
- state that is duplicated, unnecessarily derived, or owned by the wrong layer
- async, concurrency, retry, cancellation, or cleanup logic that is scattered or fragile
- performance issues that are also maintainability issues
- security-sensitive rules, such as authorization or validation, that are duplicated or inconsistently enforced
- dependency or module-boundary problems
- places where a small refactor would make future changes safer

This list is not exhaustive. If you notice any other maintainability, architecture, correctness, testability, performance, security, or code-quality issue that matters, include it.

## Judgment rules

Prefer practical, concrete recommendations.

A good finding explains where the issue is, why it matters, what could be improved later, what risk the change carries, and what tests or checks would make the refactor safer.

Avoid vague advice such as “clean this up”, “improve architecture”, “make it more modular”, or “use better abstractions”.

Do not recommend abstraction for its own sake. Sometimes a small amount of duplication is better than the wrong abstraction.

Do not recommend centralizing code unless the repeated logic genuinely represents the same concept and is likely to evolve together.

Do not recommend large rewrites when small, reviewable refactors would solve the problem.

If behavior is unclear or tests are weak, recommend characterization tests before refactoring.

## Output format

Produce this report:

# Code Inspection Report

## Scope inspected

State what you inspected, including files, directories, call sites, tests, config files, and project instruction files.

Also state anything relevant that you did not inspect.

## Executive summary

Briefly summarize the overall maintainability state.

Say whether the area appears generally healthy, moderately debt-prone, heavily debt-prone, or risky to refactor without more tests.

## Highest-value refactor opportunities

List the most important findings first.

For each finding, include:

### Finding N: Short title

- Location: specific files, functions, classes, components, modules, or patterns
- Category: duplication, centralization, complexity, dead code, architecture, tests, naming, type safety, state, async, security, performance, or other
- Severity: low, medium, or high
- Refactor risk: low, medium, or high
- Why it matters: the maintainability cost
- Suggested change: the refactor that could be done later
- Safety notes: behavior risks, tests needed, or verification steps

## Centralization candidates

List repeated concepts that may deserve a single home, such as constants, domain rules, validation logic, data mapping, types, schemas, API logic, permissions, or error handling.

For each one, suggest the most likely owner module or area.

## Possible removals

List code that appears dead, obsolete, unreachable, redundant, or misleading.

Classify each as:

- safe-looking removal
- needs verification
- do not remove without product confirmation

## Test gaps blocking safe refactors

List tests that should be added before implementation.

Focus on characterization tests that preserve current behavior before changing structure.

## Suggested refactor sequence

Suggest a safe order of operations.

Prefer small, reviewable steps.

Do not implement them.

## Things not worth changing yet

Mention code that may look imperfect but should probably not be refactored now because the duplication is acceptable, the abstraction would be premature, the behavior is unclear, the area is too risky without tests, the code is stable, or the current structure follows project conventions.

## Questions before implementation

Ask only questions that are genuinely needed before making changes.

## Final instruction

Stop after the report. Do not implement anything until the user explicitly chooses which recommendations to apply.
