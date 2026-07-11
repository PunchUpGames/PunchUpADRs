# ADR-0008: No Voluntary Inheritance

**Status:** Accepted
**Date:** 2026-06-22 (adapted for general use)
**Deciders:** Branden, Claude

## Context

Implementation inheritance is the default reuse mechanism in most object-oriented codebases, and it's the one that ages worst. Project base classes accumulate responsibilities (violating SRP), force every subclass to carry whatever the base grows (violating ISP), and couple subclasses to a base whose contract is easy to break (the recurring failure LSP exists to police). "Share behavior by subclassing a common base" is the path of least resistance and the source of a large share of long-term structural debt.

This ADR takes a stricter line than the SOLID baseline (ADR-0007) allows by default, and in doing so it changes the domain in which one SOLID letter — L — actually applies.

## Decision

**Do not create base classes to share behavior. Subclass only when a framework or other strict external dependency requires it.**

- **Share through composition, interfaces, and pure functions** — not through a common ancestor. Common behavior lives in a function (ideally a pure one in the Functional Core, ADR-0004) that multiple types call, or behind an interface that multiple types implement. It does not live in a base class they all extend.
- **No project-authored base classes.** If you find yourself writing a base class for your own code to inherit, the shared thing wants to be a function or an interface instead.
- **Framework-forced inheritance is tolerated, and only that.** When an engine, UI toolkit, or other dependency *requires* you to subclass one of its types (a controller, a widget, an entry point, a lifecycle object), do so — that's not voluntary. Keep such subclasses **thin**: a shell that adapts the framework to your composed/interface-based code, holding as little behavior as possible.

### Liskov, scoped to the carve-out

Banning voluntary inheritance makes the **Liskov Substitution Principle dormant by default** — with no voluntary subtype hierarchies, there's nothing whose substitutability could go wrong. But L is **not discarded; it is scoped.** Inside the framework-forced carve-out, you *are* inheriting, so substitutability is back in play and L is the governing rule: a required subclass must remain substitutable for its framework base without violating that base's contract. The thinness rule above is how this is upheld in practice — a subclass that adds no behavior beyond adapting to your own composed code cannot break its base's expectations.

So: L is the fallback exactly where this ADR's main rule does not reach.

## Reasons

- **Inheritance is the SOLID-violation generator.** A shared base class is the most common way SRP, ISP, and LSP get violated at once: it centralizes unrelated responsibilities, forces all of them onto every subclass, and creates a contract that's easy to break. Removing voluntary inheritance removes the generator.
- **Composition and interfaces are more flexible.** A type can compose many behaviors and implement many narrow interfaces; it can extend only one base. Favoring the former keeps types free to combine capabilities without fighting a hierarchy.
- **It strengthens I and D.** Sharing via narrow interfaces *is* Interface Segregation in practice. Sharing via abstractions a type depends on *is* Dependency Inversion. The rule pushes both SOLID letters from aspiration into default behavior.
- **It makes the L carve-out small and safe.** By confining inheritance to thin, framework-forced shells, the only places L has to be enforced are places where the subclass barely does anything — which is where substitution is hardest to get wrong.

## Consequences

### Must do

- Share behavior via composition, interfaces, and (pure) functions.
- When a framework forces a subclass, keep it thin — adapt to composed/interface-based code; minimize behavior held in the subclass.
- In any framework-forced subclass, honor LSP: remain substitutable for the base; don't override in ways that break the base's contract.

### Must not do

- No project-authored base classes intended for reuse-by-inheritance.
- No pushing shared logic "up" into a common ancestor — push it "out" into a function or "behind" an interface instead.

### Relationship to the baseline

This ADR modifies ADR-0007: it narrows **L** to the framework-forced carve-out and strengthens **I** and **D** into defaults. S and O are unaffected. The canonical statement of each letter lives in ADR-0007; this ADR holds the override.

## See Also

- ADR-0007 — SOLID as baseline design discipline (the principles this ADR narrows/strengthens; L's freestanding definition lives there).
- ADR-0004 — Functional Core / Imperative Shell (shared behavior preferentially lives as pure functions in the Core).
