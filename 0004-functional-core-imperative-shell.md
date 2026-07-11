# ADR-0004: Functional Core / Imperative Shell

**Status:** Accepted
**Date:** 2026-06-22 (consolidates a long-standing project principle into an ADR)
**Deciders:** Branden, Claude

## Context

Every codebase mixes two kinds of work: **deciding** (computing a result from inputs — rules, math, transformations) and **doing** (side effects — I/O, mutating state, talking to a framework, the network, the clock). When the two are interleaved, the deciding becomes hard to test (it can't run without the doing), hard to reason about (any line might have a hidden effect), and hard to move (logic is welded to the framework it grew up in).

The question this ADR settles is structural: where does the deciding live relative to the doing, and which way do dependencies point between them?

## Decision

Split all code into two layers with a strict one-way dependency.

### Functional Core

Pure logic lives as **immutable value types** and **stateless functions**.

- **Value types** hold state but are never mutated. Every operation returns a new copy.
- **Functions** are pure: same inputs always produce the same outputs, no side effects, no reads of ambient state (clock, RNG, global config, framework singletons).
- The Core has **no dependency on the framework, the I/O layer, or the shell**. It imports only basic types and other Core types.
- Everything here is **unit-testable in isolation** — no harness, no mocks, no world/context to stand up.

The rule of thumb: *if it thinks, decides, or computes, it belongs in the Core.*

### Imperative Shell

Everything else: state ownership, side effects, I/O, persistence, the network, the clock, and all coupling to the framework or engine. The Shell:

- Holds mutable state and performs the writes.
- Gathers inputs, calls Core functions to compute results, and applies those results to the world.
- Owns all non-determinism and all effects.

The rule of thumb: *if it touches the outside world or remembers anything between calls, it belongs in the Shell.*

### The dependency arrow points one way

```
Shell → Core        (Shell may call Core)
Core → Shell        FORBIDDEN (Core never imports, references, or calls Shell)
```

The Core does not know the Shell exists. It cannot import a Shell type, hold a reference to a Shell object, or call back into the Shell. The Shell depends on the Core; never the reverse.

### The placement test

For any new piece of code, ask: *"Does this perform a side effect, hold state across calls, read ambient/non-deterministic input, or depend on the framework?"*

- **Yes** → Shell.
- **No** → Core.

If a function is *mostly* pure but needs one effectful input (the current time, a random number, a config value), the Shell reads that input and **passes it in as an argument**. The function stays in the Core.

## Reasons

- **Testability.** Pure functions over value types are the cheapest possible thing to test: construct inputs, call, assert on the output. No setup, no teardown, no mocks. The bulk of a system's logic becomes trivially testable, and the hard-to-test part (the Shell) shrinks to thin, mostly-logicless plumbing.
- **Local reasoning.** In the Core, a function's behavior is fully determined by its arguments. You never have to ask "what else might this touch?" Effects are confined to the Shell, where you go looking for them deliberately.
- **Determinism.** A Core that can't read the clock or the RNG produces identical output for identical input, every time. That makes bugs reproducible and behavior predictable. (Enforced by the no-randomness satellite, ADR-0005.)
- **Swappable shells.** Because the Core is framework-agnostic, the same logic runs under different shells — production, tests, a different framework, a networked vs. local context — without change. The logic is portable; only the thin Shell is rewritten.
- **The boundary is where complexity is paid down.** Effects and framework coupling don't disappear — they're *concentrated* at the edge, where they're visible and contained, instead of smeared through the logic.

## Consequences

### Must do

- Core types are immutable value types; operations return new copies rather than mutating in place.
- Core functions are pure: no side effects, no ambient reads, no framework dependencies.
- Effectful inputs (time, randomness, config, external data) are read by the Shell and passed into Core functions as arguments.
- The Shell owns all state, I/O, and framework coupling; it calls the Core to decide, then applies the result.

### Must not do

- No Core file imports or references a Shell type, framework gameplay class, or I/O facility.
- No mutating methods on Core value types.
- No delegates, events, callbacks, or other effect-routing mechanisms in the Core — the Core produces *values*; the Shell routes those values to listeners.
- No non-determinism in the Core (see ADR-0005).
- No raw serialized/external data structures reaching the Core (see ADR-0006).

### Corollaries

This principle generates specific enforcement decisions, each recorded as its own ADR so it can be revisited independently:

- **ADR-0005 — No non-determinism in the Functional Core.** RNG, the clock, and frame/sequence counters live in the Shell and are passed in.
- **ADR-0006 — Parse external data at the Shell boundary.** Serialized/external data is parsed into hardened Core types at the edge; the Core never sees a raw serialized form.

Further corollaries may accrete over time; each links back here rather than being absorbed.

## See Also

- ADR-0005 — No non-determinism in the Functional Core (corollary).
- ADR-0006 — Parse external data at the Shell boundary (corollary).
- Gary Bernhardt, *Boundaries* (2012) — the talk that named the "functional core, imperative shell" pattern.
