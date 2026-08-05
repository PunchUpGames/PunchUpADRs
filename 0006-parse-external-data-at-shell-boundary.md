# ADR-0006: Parse External Data at the Shell Boundary

**Status:** Accepted
**Date:** 2026-06-22 (adapted for general use)
**Deciders:** Branden, Claude
**Applies to:** Code files

## Context

The Functional Core (ADR-0004) operates on hardened value types — structs whose fields are already the right type, already validated, already in the shape the logic expects. But data rarely *enters* a system in that shape. It arrives as serialized or external forms: config files, spreadsheets / data tables, JSON or other DTOs, database rows, API responses, framework asset objects. These forms are loosely typed, optionally-present, and untrusted — strings where you want enums, nullable fields, values that might be out of range.

If the Core consumes those forms directly, two things rot: the Core picks up a dependency on the serialization format / framework type (breaking ADR-0004's one-way arrow), and validation logic ("is this field present? in range? a known value?") smears through the pure logic that should be able to assume its inputs are already good.

This is a corollary of ADR-0004, recorded separately because it has a distinct rationale (keeping the Core dependency-free and assumption-clean) and its own revisit criteria.

## Decision

**Parse external/serialized data into hardened Core types at the Shell boundary. The Core never sees a raw serialized or framework-native form.**

The Shell owns a thin **parse-and-harden** step at the edge: it reads the loose external form (config row, DTO, asset object, API payload), validates it, and converts it into a Core value type with the right field types and guaranteed invariants. From that point inward, the Core works only with the hardened type and may *assume its inputs are valid* — no defensive re-checking, no null-handling, no "what if this enum string is unknown" branching in the logic.

This is the "**parse, don't validate**" discipline: turn untrusted input into a type that *cannot* represent the invalid states, once, at the boundary — rather than validating the same loose value repeatedly everywhere it's used.

## Reasons

- **Keeps the Core's one-way dependency intact.** The Core never imports the serialization library, the data-table type, or the framework asset class. Only the Shell's parse step touches those. The Core stays portable and framework-agnostic per ADR-0004.
- **Validation happens once, at the edge.** Instead of every Core function defending against malformed input, a single boundary step rejects or normalizes it. The Core's type signatures become honest: if a function takes a `HardenedThing`, a `HardenedThing` is all it can receive.
- **Failures surface early and in one place.** A missing field or bad value is caught at parse time, at the boundary, with full context about *which* external record was wrong — not deep inside the logic as a confusing downstream symptom.
- **The Core stays assumption-clean.** Pure logic that can assume its inputs are well-formed is simpler, smaller, and easier to test — no branches exist for states the type makes impossible.

## Consequences

### Must do

- Define hardened Core value types that represent only valid states.
- Put a parse-and-harden step in the Shell at every point external data enters: config/asset load, deserialization, API ingestion, DB read.
- Validate and convert there; reject or normalize bad input at that boundary.
- Pass only hardened Core types inward.

### Must not do

- No Core code that imports or references a serialization format, data-table/asset type, or DTO.
- No Core function that re-validates input the boundary already guaranteed.
- No raw external form (loose dict/row/payload, framework asset object) passed into the Core.

### Testing

- The parse-and-harden step is Shell code and is tested at the boundary: valid input → correct hardened type; malformed input → rejected/normalized as specified.
- Core functions are tested against hardened types directly, with no malformed-input cases (those can't reach them by construction).

## Validation

- Trace a new external-data path (config load, API response, save-file deserialize) end to end: is there exactly one parse-and-harden step, and does every Core-layer function on that path receive only the hardened type it produces?
- Grep Core-layer files for a type name matching the project's serialization/DTO/asset types (a JSON row type, an ORM model, a scene/resource class) — presence there means raw external data reached the Core.
- Example naming a language: in Python, a Core function accepting a raw `dict` fresh from `json.load()` instead of a typed/validated dataclass is the pattern this ADR forbids.

## See Also

- ADR-0004 — Functional Core / Imperative Shell (the parent principle this enforces).
- ADR-0005 — No non-determinism in the Functional Core (sibling corollary).
- Alexis King, *Parse, Don't Validate* (2019) — the essay that named the boundary-parsing discipline.
- ADR-0016 — ADR authoring conventions (defines the `Applies to:` tag and `## Validation` section used here).
