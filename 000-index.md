# Punch Up Games — Studio ADR Index

Architecture Decision Records that apply to all Punch Up Games projects. These capture decisions about how we work, what we build on, and how we design — independent of any specific game engine or project.

Numbers are permanent identifiers assigned once and never changed. Reading order and grouping are here, not in the filenames.

**If you're new here:** start with the Values & Working Norms cluster below. Those three ADRs set the context for everything else.

---

## How to use this index (for AI contributors)

Every ADR carries an **Applies to** tag (see ADR-0016) — `Always`, `Code files`, `Test files`, `Plan files`, or `Documentation`. Use it to decide what to load, rather than reading every body on every task:

1. **`Always` ADRs are read in full every time the ADR gate fires**, regardless of what the current task touches.
2. **For every other ADR, read the full body only if its tag matches a category the current task touches.** If nothing in the task matches, this table's one-line summary is enough — no need to open the body.
3. **An ADR with no tag yet (`—` below) has not been through the ADR-0016 pass. Treat it as `Always` until it's tagged** — an untagged ADR is unclassified, not exempt. Don't silently skip it.
4. **At planning time, when the task's file categories aren't fully known yet, err toward reading the full body.** A body is 50–100 lines; a wrong architectural call downstream costs far more.
5. **Each ADR's `## Validation` section is what to consult during implementation and review** — it gives concrete example checks so compliance doesn't have to be re-derived from the principle every time.

This same convention (tag + Validation section) applies to project-local ADR indexes, not just this studio one.

---

## Values & Working Norms

These establish who we are as contributors and how we work together. Read them before the technical decisions.

| # | Title | Applies to | One line |
|---|---|---|---|
| [0001](0001-developer-tooling-is-a-personal-choice.md) | Developer Tooling is a Personal Choice | Always | No tools are required or assumed — AI use is one valid approach among several |
| [0002](0002-ai-collaboration-norms.md) | AI Collaboration Norms | Always | How AI and human contributors work as peers, with the human as final authority |
| [0003](0003-documentation-standards.md) | Documentation Standards | Documentation | Where docs live, how they're shaped, and when AI may modify them |
| [0015](0015-the-three-virtues.md) | The Three Virtues of a Programmer | Always | Laziness, Impatience, Hubris — Larry Wall's framework, with behavioral interpretation |
| [0016](0016-adr-authoring-conventions.md) | ADR Authoring Conventions | Documentation | Defines the `Applies to` tag and `## Validation` section used throughout this index |

---

## Architecture

The technical decisions governing how codebases are structured. These are the north star for new code; see ADR-0014 for how they apply to legacy code.

| # | Title | Applies to | One line |
|---|---|---|---|
| [0004](0004-functional-core-imperative-shell.md) | Functional Core / Imperative Shell | Code files | Pure logic in the core; all state, I/O, and framework coupling in the shell |
| [0005](0005-no-nondeterminism-in-functional-core.md) | No Nondeterminism in Functional Core | Code files | The core produces identical outputs for identical inputs — always |
| [0006](0006-parse-external-data-at-shell-boundary.md) | Parse External Data at Shell Boundary | Code files | Validate and transform untrusted data before it reaches the core |
| [0007](0007-solid-baseline.md) | SOLID Baseline | Code files | Single responsibility, open/closed, Liskov, interface segregation, dependency inversion |
| [0008](0008-no-voluntary-inheritance.md) | No Voluntary Inheritance | Code files | Prefer composition; use inheritance only when the type relationship is genuinely is-a |
| [0009](0009-nothing-is-something-null-object.md) | Nothing is Something (Null Object) | Code files | Replace null checks with objects that express the "nothing" case explicitly |
| [0010](0010-tdd-discipline.md) | TDD Discipline | Code files, Test files | Write the test first; let failing tests drive implementation |
| [0011](0011-tests-are-append-only.md) | Tests Are Append-Only | Test files | Tests are never modified or deleted — only added |
| [0012](0012-authority-driven-events.md) | Authority-Driven Events | Code files | Events flow from an authoritative source; side effects react, they don't rebroadcast |

---

## Operational Policy

How the above applies in practice — permissions, legacy code, and the mechanics of the working day.

| # | Title | Applies to | One line |
|---|---|---|---|
| [0013](0013-ai-permissions.md) | AI Permissions | Always | What the AI agent may do without prompting, and what always requires a human check |
| [0014](0014-legacy-code-and-transition-policy.md) | Legacy Code and Transition Policy | Code files, Plan files | ADRs are the north star for new code; existing code is in transition, not violation |
