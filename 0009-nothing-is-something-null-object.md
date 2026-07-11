# ADR-0009: Nothing Is Something — Model Absence as a Null Object

**Status:** Accepted
**Date:** 2026-06-22 (adapted for general use)
**Deciders:** Branden, Claude

## Context

The most common conditional in a codebase is the absence check: "is this thing here?" — `if x is null`, `if list is empty`, `if not found`. Each such check is a branch, and the branch tends to *spread*: every function that touches a possibly-absent value has to ask the same question and handle the same "not there" case. The absence becomes everyone's problem, repeated everywhere, and the real logic gets buried under guard clauses.

The insight (Sandi Metz, *Nothing Is Something*, RailsConf 2015): **the absent / empty / missing case is part of the domain, so model it as a real thing** — an object or value that honors the same interface as the "present" case and does the sensible nothing. Then callers stop asking "is it there?" because there's always *something* there to talk to, and that something behaves correctly.

This is unrecorded in our ADRs; the nearest neighbors are ADR-0006 (reject invalid input at the boundary) and the polymorphism-over-conditionals thread in ADR-0007/0008, but the Null Object principle itself is distinct.

## Decision

**Represent absence as a first-class value or object that honors the same interface as the present case, instead of using null + absence-checks.** Callers operate on it uniformly; the "nothing" handles itself.

The principle takes two idioms across the architecture:

### In the Functional Core — absence is an empty/identity *value*

The null case is a value of the same type (or a sibling in the same closed set) that the pure functions handle as a matter of course — an empty collection, an identity element that folds to "no change," a typed "absent" value. Functions don't branch on it; they apply the same operation and it naturally produces the right result. This removes the absence-check from *every* Core function that could otherwise receive a null — the check that would otherwise be copy-pasted across the whole layer simply doesn't exist.

There can be **more than one kind of nothing.** A closed set of special-case values (e.g. distinct "empty" vs. "rejected" cases) is fine and often clearer than a single null, provided each one is substitutable and the functions treat the whole set uniformly.

### In the Imperative Shell — absence is a no-op *object*

The null case is an object implementing the same interface with do-nothing / sensible-default behavior — a logger that discards, a handler that ignores, a step that passes through. The Shell wires it in like any other implementation, and downstream code calls it without checking, getting a clean no-op instead of a branch.

### Substitutability is the load-bearing requirement

A Null Object only earns its keep if it is **fully substitutable** for the real one — it must honor the *entire* contract callers rely on. If callers still have to special-case it for some messages, the branch is back and the pattern has failed. This is the Liskov Substitution Principle (ADR-0007) doing exactly the job it exists for; it's also why the closed set of "nothings" must each satisfy the same interface.

## When NOT to use this

A Null Object is for **legitimate absence with a genuinely correct default behavior** — not a tool for swallowing errors that should fail loudly. A "nothing" that silently absorbs a state which *should* have stopped the program is a bug-hider, and that cuts against both honest failure (ADR-0001) and reject-at-the-boundary (ADR-0006).

The two ADRs cooperate rather than conflict: **ADR-0006 governs invalid states; ADR-0009 governs valid-but-empty ones.**

- Invalid / malformed input is still detected and rejected at the boundary per ADR-0006.
- A legitimate boundary normalization is to *log the problem and emit a well-typed "nothing"* that flows onward harmlessly — a value that can be trivially ignored or processed as a no-op, never one that triggers a crash or a downstream gate. That converts a detected error into a safe, in-domain absence on purpose; it does not skip the detection.

The test: if the "nothing" represents *"this legitimately isn't here, and doing nothing is correct,"* use a Null Object. If it represents *"something is wrong,"* fail at the boundary instead.

## Reasons

- **It deletes the most-repeated conditional.** The absence-check otherwise appears in every consumer of a nullable value. Modeling absence as a behaving thing removes the check from all of them at once.
- **The real logic stays visible.** Functions express what they *do*, not a preamble of guards against what might be missing.
- **It composes with the rest of the library.** In the Core it's just another immutable value (ADR-0004); the uniform-handling it enables is the same instinct as making illegal states unrepresentable (ADR-0006); its substitutability requirement is LSP (ADR-0007).
- **Multiple "nothings" scale better than nested nulls.** A closed set of substitutable special-case objects beats a single null that every site must re-interpret.

## Consequences

### Must do

- Model the empty/absent/missing case as a value or object that honors the present case's interface.
- In the Core, give pure functions an empty/identity value to fold over instead of a null to branch on.
- In the Shell, provide a no-op implementation of the interface instead of null + caller-side checks.
- Ensure every null object (and every member of a special-case set) is fully substitutable for the real thing.

### Must not do

- No scattering of absence-checks across consumers when a Null Object would remove them.
- No Null Object that fails substitutability — if callers must special-case it, it isn't one.
- No using a Null Object to swallow a genuine error; reject invalid states at the boundary (ADR-0006) and fail honestly (ADR-0001).

## See Also

- Sandi Metz, *Nothing Is Something*, RailsConf 2015 — <https://www.youtube.com/watch?v=29MAL8pJImQ>.
- ADR-0004 — Functional Core / Imperative Shell (the null case is an immutable Core value or a Shell object).
- ADR-0006 — Parse external data at the Shell boundary (governs invalid states; this ADR governs valid-but-empty ones; the two cooperate at the boundary).
- ADR-0007 — SOLID baseline (Liskov Substitution is what makes a Null Object safe).
