# ADR-0011: Tests Are Append-Only

**Status:** Accepted
**Date:** 2026-06-22 (adapted for general use)
**Deciders:** Branden, Claude
**Applies to:** Test files

## Context

The standard TDD loop (ADR-0010) says how a test is *born*: red, then green. It says nothing about what happens to that test over the months that follow, as the code around it changes. The default, undisciplined answer is that tests churn — a method's signature changes, and a swathe of tests gets rewritten to match. That churn is corrosive: rewritten tests no longer prove the behavior they originally pinned, every rewrite is a chance to silently weaken an assertion, and a large diff of "updated tests" hides whether any guarantee actually changed.

This project takes a stronger line, made possible by the rest of the library: the Functional Core is immutable value types and pure functions (ADR-0004), behavior is composed rather than inherited (ADR-0008), and code is open for extension but closed for modification (ADR-0007, O). Under those conditions, a test almost never *needs* to change — and if it seems to, that's a signal pointing at a better move.

## Decision

**Once written, a test is almost never modified.** The test corpus accretes; it does not churn. A test changes only if the test itself was buggy, or the *requirement it encodes* genuinely changed — and a changed requirement is usually a new function, not an edited one.

### When implementation "needs" a test change, write a new function instead

If a Functional Core function diverges enough that its existing tests would have to change — a parameter added or removed, a changed return shape, a different contract — the move is **not** to edit the function and its tests in place. It is to write a **new function with new tests**, and let the old and new **compose**:

- the new function calls the old one within its implementation, or
- the old function becomes a special-case configuration of the new one (old delegates to new with fixed arguments).

Either way, the old function keeps its behavior and its tests keep passing, untouched. The new behavior gets new tests, born red-then-green per ADR-0010. The corpus grew; nothing was rewritten.

### The anti-pattern this forbids

> *"I had to change this method, and now I have to change all of these tests."*

If you find yourself there, stop: that is the signal to introduce a new function and compose, not to rewrite the test suite. The need to change many tests at once is almost always evidence that a new function is wanted — the existing one is being asked to become something it wasn't.

### The narrow exceptions

A test *is* edited only when:

- **the test was wrong** — it asserted the wrong thing, or never actually exercised its path (and is being corrected, not relaxed); or
- **the requirement genuinely changed** and the old behavior is being *retired*, not extended — in which case the old function and its now-obsolete tests are removed together, deliberately, not silently mutated.

## Reasons

- **It is Open/Closed applied to functions and their tests.** ADR-0007's O says extend by adding, not by modifying. A function plus its tests is a unit of behavior; you extend the system by adding a new one, leaving the proven one intact.
- **An unchanged test keeps proving exactly what it always proved.** A test that is never rewritten is a stable, trustworthy guarantee. A rewritten test only proves whatever it asserts *now*, and the rewrite is an unaudited opportunity to weaken it.
- **Composition makes it cheap.** Because behavior is composed (ADR-0008) and Core functions are pure (ADR-0004), wrapping an old function in a new one — or expressing the old as a configuration of the new — is a natural, low-friction move rather than a structural fight.
- **The "must change many tests" smell is information.** It reliably indicates that a function is being overloaded into a second responsibility. Honoring it produces a new, well-named, well-tested function instead of a quietly broadened old one.

## Consequences

### Must do

- Treat existing tests as fixed. Add tests; don't rewrite them.
- When a Core function would need a signature/contract change, add a new function with new tests and compose old and new (one delegates to the other).
- When retiring behavior, remove the old function and its tests together, deliberately.

### Must not do

- No editing a method and then updating its tests to match. That is the forbidden pattern.
- No relaxing or rewriting an assertion to make a changed implementation pass.
- No silent mutation of a test to track a requirement change — retire-and-replace instead.

## Validation

- Diff any modified test file: is an existing `test_*` (or language-equivalent test function) body changed, weakened, or removed, rather than a new one added? That's presumptively a violation unless the commit or PR explicitly states either "this test was wrong" or "this requirement was retired" (the two named exceptions).
- Watch for the smell named in Context directly: a commit message or session transcript containing "had to update these tests to match" is close to a direct admission of the forbidden pattern — check whether an additively-composed new function would have let the original tests keep passing untouched.
- Example, naming a language explicitly: in GDScript or C#, a signature change to an existing function that forces edits across many of its existing test cases is the trigger to stop and ask whether a new function (old delegates to new, or new wraps old with fixed arguments) avoids the rewrite.

## See Also

- ADR-0010 — TDD discipline (how a test is born; this ADR governs how the corpus evolves afterward).
- ADR-0007 — SOLID baseline (this is the Open/Closed principle applied to functions and their tests).
- ADR-0008 — No Voluntary Inheritance (composition is what makes new-function-over-edit cheap).
- ADR-0004 — Functional Core / Imperative Shell (immutable values + pure functions make tests stable in the first place).
- ADR-0016 — ADR authoring conventions (defines the `Applies to:` tag and `## Validation` section used here).
