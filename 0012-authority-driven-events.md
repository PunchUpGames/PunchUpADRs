# ADR-0012: Authority-Driven Events

**Status:** Accepted
**Date:** 2026-06-22 (adapted for general use)
**Deciders:** Branden, Claude

## Context

When state changes, other parts of the system — UI, logging, dependent components — need to know. The Functional Core produces new values (ADR-0004); the Shell owns the state and is responsible for routing those values to whoever is listening. *How* the Shell does that routing is the decision here.

Two broad approaches exist:

1. **Change detection (bottom-up).** Wrap state values in smart objects that track whether they changed. Listeners poll the wrapper or subscribe to a generic "something changed" signal. The wrapper drives notification.
2. **Authority-driven events (top-down).** The code that *causes* the change fires an explicit event at the point of change. Listeners react to it. State fields stay plain types.

Change detection is the common fit for systems where mutations happen in many places and no single owner controls writes — reactive UI frameworks, shared attribute systems, generic pub/sub. This project is not that system: the Functional Core / Imperative Shell split (ADR-0004) plus single-responsibility (ADR-0007) mean there is always exactly one owner that writes any given piece of state.

## Decision

**Use authority-driven events.** State fields are plain values. The component that owns and writes a piece of state fires an explicit event immediately after the write. Listeners subscribe to that event; they never poll and never wrap.

### Events carry meaningful context, not just "it changed"

The event the authority fires names *what happened* and carries the relevant data — the new value, the delta, the reason/source — not a bare "value changed" tick. `OnDamageApplied(amount, newTotal, source)` is worth more to every listener than "health changed," and it costs nothing extra because the authority already knows all of it at the write site.

### State rides on the event payload, not on a parallel channel

When a listener needs a *new* piece of state, add it to the payload the existing event already delivers — do not stand up a second notification channel for it. The event surface already crosses from authority to listener; a new field on that payload reaches every listener for free, with no new subscription, no new wiring, and no per-listener plumbing. A proliferation of narrow parallel events for individual bits is the smell this avoids.

(This pairs with the Null Object discipline, ADR-0009: a listener filters the payload's state — empty / active / dead — without branching, because the "nothing" cases are themselves well-typed values on the payload.)

## Reasons

- **The authority already knows.** With a single write owner (ADR-0004, ADR-0007), there's no ambiguity about who changed what — the writer is right there. It can fire a precise event with full context. Detection machinery would only be re-discovering what the writer already had in hand.
- **Zero cost when nothing happens.** Change detection compares old-vs-new on every potential mutation, even when nothing changed. An event you simply don't fire costs nothing.
- **Explicit beats inferred.** A named event with a reason is more useful to listeners and easier to reason about than a generic "changed" signal they must interpret.
- **Payload-borne state scales; parallel channels don't.** One event surface carrying richer payloads stays simple as needs grow; a new delegate per state bit multiplies wiring and per-listener bindings.

## Consequences

### Must do

- State fields on Shell components are plain values.
- The owner that writes state fires an explicit event right after the write.
- Events carry meaningful context (value, delta, reason/source).
- New state a listener needs goes on the existing event's payload when one exists.

### Must not do

- No wrapping plain state fields in change-tracking objects.
- No generic "attribute changed" subscription infrastructure, and no polling for changes.
- No new parallel notification channel for a bit that could ride an existing payload.

### When change detection would come back

If the system ever needed *generic, reusable listeners that don't know what they're bound to* (e.g. a widget that renders any two numbers as a percentage across unrelated systems), a keyed-subscription mechanism would be the tool. That is a different problem; for a system with clear write authorities, direct event firing is simpler and cheaper.

## See Also

- ADR-0004 — Functional Core / Imperative Shell (the Core produces values; this ADR is how the Shell routes them to listeners).
- ADR-0007 — SOLID baseline (single write authority is Single Responsibility applied to state ownership).
- ADR-0009 — Nothing Is Something (listeners filter payload-borne state without branching).
