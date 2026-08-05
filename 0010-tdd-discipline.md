# ADR-0010: Test-Driven Development Discipline

**Status:** Accepted
**Date:** 2026-06-22 (adapted for general use)
**Deciders:** Branden, Claude
**Applies to:** Code files, Test files

## Context

Test-driven development is a well-documented practice with a deep literature; this ADR does not re-teach it. It records *the decision to adopt TDD as the project's default discipline* and the few clauses where this project pins down specifics — chiefly **what** gets tested, which follows from the Functional Core / Imperative Shell split (ADR-0004). The canonical depth is deferred to the literature (see *See Also*).

## Decision

Adopt TDD as the default for all logic that is unit-testable. Three clauses:

### 1. Scope — the Functional Core is thoroughly unit-tested; the Imperative Shell is not

- **Functional Core: tested exhaustively.** Pure functions over immutable value types are the cheapest possible thing to test — construct inputs, call, assert on outputs. The Core is where the logic lives, and it's where the tests live. This is the payoff ADR-0004 was designed to produce.
- **Imperative Shell: not unit-tested.** The Shell is thin, effectful plumbing — I/O, state, framework coupling. A "unit test" of Shell code mostly exercises mocks and the framework, not project logic. Effort spent there buys little. Confidence in the Shell comes from keeping it thin and from higher-level checks (manual, integration), not from unit tests.

The practical consequence: as the Core grows, its tests grow with it; the Shell stays lean enough not to need them.

### 2. Test-first, red before green

Write the failing test first. **Observe it fail.** Then write the implementation that turns it green. A test that has never been seen failing proves nothing — it might pass for reasons unrelated to the behavior it claims to check.

On coincidentally-green tests: if a newly written test passes *before* its implementation exists, that is a **signal to investigate**, not something to accept silently. Either the behavior already exists (so this test isn't driving any new work and may belong elsewhere), or the test isn't actually exercising the intended path (it's miswired). Resolve which before moving on.

This is a **norm, not a mandated ritual.** The rule is "see it fail; if it won't, find out why." Deliberately inverting an assertion to prove a test *can* fail is a useful tool when you genuinely doubt a test exercises its path — keep it available, but it is not required on every test.

### 3. Bug fixes start with a red test

A bug fix begins by writing a test that reproduces the bug against the current (broken) code — so it fails for the right reason — and only then fixing the code to turn it green. This guarantees the test actually captures the bug and guards against its return. A regression test that was never red is not known to guard anything.

## Reasons

- **The scope rule concentrates effort where it pays.** ADR-0004 made the Core cheap to test and the Shell mostly logicless. Testing the Core exhaustively and skipping Shell unit tests spends effort exactly where it converts to confidence.
- **Red-before-green proves the test works.** The failure is evidence that the test exercises the behavior and that the implementation is what flips it. Without the observed failure, a passing test is unverified.
- **Bug-first-red prevents silent regressions.** A test written after the fix, never having failed, can pass for the wrong reason and let the bug return undetected.

## Consequences

### Must do

- Write unit tests for all Functional Core logic; aim for thorough coverage there.
- Write the test before the implementation; observe it fail before making it pass.
- Begin every bug fix with a failing test that reproduces the bug.
- Investigate any test that is green before its implementation exists.

### Must not do

- No unit tests standing up the Imperative Shell with mocks to test framework behavior.
- No implementation written ahead of its test.
- No regression test that was never observed red.

### Related discipline

How the test corpus *evolves* over time — and why tests are almost never modified once written — is its own decision: see ADR-0011 (Tests Are Append-Only).

## Validation

- Check that a new Core-layer function shipped with tests, and that no new unit test exists purely to stand up Shell plumbing with mocks.
- For a bug-fix diff: is there a test that reproduces the bug? This one can't be verified from the diff alone — check the session transcript or commit history for the test failing against pre-fix code before the fix landed.
- Ask directly rather than inferring: "was this test observed failing before the implementation existed?" If a test and its implementation appear in the same step with no failing run in between, treat it as unverified, not compliant.

## See Also

- Kent Beck, *Test-Driven Development by Example* — the canonical practice this ADR adopts and defers to.
- ADR-0004 — Functional Core / Imperative Shell (the scope rule is a direct corollary).
- ADR-0011 — Tests Are Append-Only (how the test corpus grows without churning).
- ADR-0016 — ADR authoring conventions (defines the `Applies to:` tag and `## Validation` section used here).
