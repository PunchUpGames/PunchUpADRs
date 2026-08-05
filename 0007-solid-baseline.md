# ADR-0007: SOLID as Baseline Design Discipline

**Status:** Accepted
**Date:** 2026-06-22 (adapted for general use)
**Deciders:** Branden, Claude
**Applies to:** Code files

## Context

A project needs a shared, named vocabulary for object/module design — a default discipline that contributors (human and AI) can assume is in force without re-arguing it on every decision. SOLID is the obvious choice: it's well-defined, heavily documented, has decades of educational material, and is widely understood. Adopting it as the baseline means most design discussions can reference a letter instead of re-deriving a principle.

This ADR records the *decision to adopt SOLID as the baseline* and — importantly — flags where this project's other ADRs **go further than, narrow, or override** a given letter. SOLID is the floor; the deltas sit on top of it and point back down here.

## Decision

**Adopt SOLID as the default design discipline for all code.** The five principles, stated compactly (the canonical literature carries the depth — see *See Also*):

- **S — Single Responsibility.** A module/class/function has one reason to change. If describing it requires "and," split it.
- **O — Open/Closed.** Open for extension, closed for modification. Prefer adding a new implementation behind an abstraction over editing existing code with new branches.
- **L — Liskov Substitution.** A subtype must be usable anywhere its base type is expected, without violating the base's contract.
- **I — Interface Segregation.** Many small, client-specific interfaces beat one fat interface. A consumer shouldn't depend on methods it doesn't use.
- **D — Dependency Inversion.** High-level policy depends on abstractions, not on low-level details. Details depend inward, toward the policy.

### Where downstream ADRs modify the baseline

This project does not take all five letters at their textbook face value. Two carry deltas:

- **L is narrowed by ADR-0008 (No Voluntary Inheritance).** Because implementation inheritance is disallowed except where a framework forces it, L is *dormant by default* — there are no voluntary subtype hierarchies to get substitution wrong. L remains **authoritative inside the carve-out**: where inheritance is unavoidable, substitutability is exactly the governing rule. See ADR-0008.
- **D is realized architecturally by ADR-0004 (Functional Core / Imperative Shell).** The one-way dependency arrow (Shell → Core, never reverse) *is* Dependency Inversion at the architecture scale: high-level logic (the Core) depends on nothing; the framework-coupled details (the Shell) depend inward toward it. ADR-0004 is where D's teeth live.

S, O, and I apply as written, with no project-specific override.

## Reasons

- **A named baseline removes per-decision overhead.** "This violates SRP" is a complete argument; everyone knows the principle. Without an adopted baseline, every design review re-litigates first principles.
- **It's the fallback when a stronger local rule is worked around.** This project's stricter rules (no voluntary inheritance, FC/IS) have carve-outs. When you fall *out* of a stricter rule, you fall *back onto* SOLID. The baseline has to exist as a freestanding, documented thing for the fallback to mean anything — which is the whole reason this ADR precedes the deltas that modify it.
- **It's externally documented, so this ADR stays thin.** Per the documentation standards (concise; don't restate what's well-covered elsewhere), this ADR states the decision and the deltas, and defers the teaching to canonical sources.

## Consequences

- SOLID is assumed in force for all code unless a more specific ADR overrides a given letter (currently L via ADR-0008, D via ADR-0004).
- Design discussions may reference letters as shorthand; "fix the SRP violation" is an actionable instruction.
- When a new ADR strengthens, narrows, or overrides a SOLID letter, it cites this ADR (delta → base), and a one-line "narrowed/realized by ADR-N" pointer is added here (base → delta). The authoritative statement lives in the delta; this ADR only indexes it.

## Validation

No single check covers all five letters; SOLID stays a judgment call even where the deltas below are mechanical. Probes worth asking per letter on a new class/module:

- **S:** does describing its responsibility need "and"? That's the split signal.
- **O:** does adding a new case require editing an existing conditional/switch, or does it add a new implementation behind an existing abstraction?
- **I:** does a new interface method force an implementer that doesn't need it to add a no-op or throw-not-implemented stub?

For **L** and **D**, check the delta ADRs first rather than re-deriving here — see below.

## See Also

- ADR-0008 — No Voluntary Inheritance (narrows L; strengthens I and D — has L's concrete validation checks).
- ADR-0004 — Functional Core / Imperative Shell (realizes D as the one-way dependency arrow — has D's concrete validation checks).
- Robert C. Martin, *Agile Software Development: Principles, Patterns, and Practices* — origin of the SOLID grouping; the canonical depth this ADR defers to.
- ADR-0016 — ADR authoring conventions (defines the `Applies to:` tag and `## Validation` section used here).
