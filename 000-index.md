# Punch Up Games — Studio ADR Index

Architecture Decision Records that apply to all Punch Up Games projects. These capture decisions about how we work, what we build on, and how we design — independent of any specific game engine or project.

Numbers are permanent identifiers assigned once and never changed. Reading order and grouping are here, not in the filenames.

**If you're new here:** start with the Values & Working Norms cluster below. Those three ADRs set the context for everything else.

---

## Values & Working Norms

These establish who we are as contributors and how we work together. Read them before the technical decisions.

| # | Title | One line |
|---|---|---|
| [0001](0001-developer-tooling-is-a-personal-choice.md) | Developer Tooling is a Personal Choice | No tools are required or assumed — AI use is one valid approach among several |
| [0002](0002-ai-collaboration-norms.md) | AI Collaboration Norms | How AI and human contributors work as peers, with the human as final authority |
| [0003](0003-documentation-standards.md) | Documentation Standards | Where docs live, how they're shaped, and when AI may modify them |

---

## Architecture

The technical decisions governing how codebases are structured. These are the north star for new code; see ADR-0014 for how they apply to legacy code.

| # | Title | One line |
|---|---|---|
| [0004](0004-functional-core-imperative-shell.md) | Functional Core / Imperative Shell | Pure logic in the core; all state, I/O, and framework coupling in the shell |
| [0005](0005-no-nondeterminism-in-functional-core.md) | No Nondeterminism in Functional Core | The core produces identical outputs for identical inputs — always |
| [0006](0006-parse-external-data-at-shell-boundary.md) | Parse External Data at Shell Boundary | Validate and transform untrusted data before it reaches the core |
| [0007](0007-solid-baseline.md) | SOLID Baseline | Single responsibility, open/closed, Liskov, interface segregation, dependency inversion |
| [0008](0008-no-voluntary-inheritance.md) | No Voluntary Inheritance | Prefer composition; use inheritance only when the type relationship is genuinely is-a |
| [0009](0009-nothing-is-something-null-object.md) | Nothing is Something (Null Object) | Replace null checks with objects that express the "nothing" case explicitly |
| [0010](0010-tdd-discipline.md) | TDD Discipline | Write the test first; let failing tests drive implementation |
| [0011](0011-tests-are-append-only.md) | Tests Are Append-Only | Tests are never modified or deleted — only added |
| [0012](0012-authority-driven-events.md) | Authority-Driven Events | Events flow from an authoritative source; side effects react, they don't rebroadcast |

---

## Operational Policy

How the above applies in practice — permissions, legacy code, and the mechanics of the working day.

| # | Title | One line |
|---|---|---|
| [0013](0013-ai-permissions.md) | AI Permissions | What the AI agent may do without prompting, and what always requires a human check |
| [0014](0014-legacy-code-and-transition-policy.md) | Legacy Code and Transition Policy | ADRs are the north star for new code; existing code is in transition, not violation |
