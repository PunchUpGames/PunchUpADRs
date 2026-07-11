# ADR-0005: No Non-Determinism in the Functional Core

**Status:** Accepted
**Date:** 2026-06-22 (adapted for general use)
**Deciders:** Branden, Claude

## Context

The Functional Core (ADR-0004) is defined by purity: same inputs, same outputs, every time. The most common way that property gets silently broken is not a side effect — it's an *ambient read* of a non-deterministic source: a random number, the wall clock, a frame or sequence counter, a process-unique id. Code that calls `random()` or `now()` deep inside an otherwise-pure function looks pure at the call site but isn't: it returns different results on identical inputs, and it can't be tested by asserting on outputs.

This is a corollary of ADR-0004, recorded separately because it has its own enforcement surface and could be loosened independently (e.g., a project might one day allow a seeded PRNG inside the Core) without abandoning the Core/Shell split.

## Decision

**The Functional Core must not invoke any source of non-determinism.** Specifically, no Core code may call:

- a random number generator,
- the system clock / wall time / monotonic time,
- a frame counter, tick number, or sequence counter that advances on its own,
- any other source whose value depends on *when* or *where* it runs rather than on the arguments passed in.

When a Core computation needs such a value, **the Shell reads it and passes it in as an argument.** The randomness, the timestamp, the frame number become ordinary inputs. The Core function that consumes them stays pure: given the same seed / timestamp / frame, it produces the same result.

## Reasons

- **Reproducibility.** A Core that can't read hidden state produces identical output for identical input. Bugs reproduce on the first try; you never chase "it only fails sometimes."
- **Testability.** A function that takes its randomness as a parameter can be tested with a fixed seed and asserted exactly. A function that calls the RNG internally can only be tested statistically, which is slow, flaky, and weak.
- **The Shell is the honest home for "when" and "how much."** Deciding *which* random value or *what* timestamp to use is itself a decision the Shell should own and be able to vary (record/replay, deterministic simulation, networked lockstep, test fixtures). Passing it in keeps that decision visible at the boundary.
- **It generalizes for free.** Seeded simulation, replay systems, and deterministic networking all fall out of "the Core is a pure function of its inputs, including its randomness." You get them without extra machinery once the RNG is an argument rather than an ambient call.

## Consequences

### Must do

- Generate random values, read the clock, and read frame/sequence counters in the Shell.
- Pass those values into Core functions as explicit arguments.
- For anything seedable, thread the seed through as data so the same seed reproduces the same run.

### Must not do

- No RNG calls inside Core code.
- No clock / time reads inside Core code.
- No frame-counter or auto-advancing-counter reads inside Core code.

### Testing

- Core functions that consume randomness or time are tested with fixed, supplied values and exact assertions — never with statistical tolerances.

## See Also

- ADR-0004 — Functional Core / Imperative Shell (the parent principle this enforces).
- ADR-0006 — Parse external data at the Shell boundary (sibling corollary; both keep the Core a pure function of explicit inputs).
