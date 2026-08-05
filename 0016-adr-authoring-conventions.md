# ADR-0016: ADR Authoring Conventions

**Status:** Proposed
**Date:** 2026-08-05
**Deciders:** Branden, Claude
**Applies to:** Documentation

## Context

As the ADR corpus has grown, informal review has surfaced a pattern: ADRs framed as a single narrow, checkable rule (append-only tests, TDD's red-before-green, runtime `load()` for cyclic scenes) get cited and followed in real design decisions measurably more than ADRs framed as architecture-wide principles (no voluntary inheritance, null objects, authority-driven events) — not because the principles are wrong, but because "hold this principle in mind at all times" competes for a shrinking budget of attention against everything else a contributor (human or AI) is holding at once. Every added page of always-on guidance makes every other page slightly less likely to survive contact with a real task.

Two structural gaps compound this:

1. **No cheap way to know which ADRs are relevant to the file about to be touched.** A reader either re-reads the whole corpus per task (expensive, and the volume problem above) or relies on a single planning-time read (lossy — the details fade well before implementation finishes).
2. **Even an ADR held in mind offers no concrete compliance check.** A principle like "no voluntary inheritance" has to be freshly re-derived into "is *this* diff okay?" every time — exactly the kind of derivation that gets skipped under time pressure.

## Decision

Every ADR in this corpus carries two additions to the existing Status/Date/Deciders/Scope → Context → Decision → Reasons → Consequences → See Also shape.

### 1. An `Applies to:` tag in the metadata header

Placed alongside Status/Date/Deciders/Scope. Value is one or more of a fixed, small vocabulary — deliberately **not** a file extension or language, since this corpus is read by projects that don't share an engine or language:

| Tag | Meaning |
|---|---|
| `Always` | A working norm or standing behavioral expectation, not tied to a category of artifact. Read in full whenever the ADR gate fires. |
| `Code files` | Governs the shape of source code. |
| `Test files` | Governs how tests are written or evolved. |
| `Plan files` | Governs decisions made during planning, before implementation. |
| `Documentation` | Governs how docs, comments-as-docs, or the ADRs themselves are written. |

An ADR may carry more than one tag. The tag is a *loading trigger* for a reader deciding what to open, not a restriction on what the ADR covers — see the index's usage instructions for how it drives that decision, including the required default for ADRs not yet tagged.

### 2. A `## Validation` section, placed after Consequences and before See Also

A short set of concrete, illustrative checks a reviewer — human or AI — can run against a specific diff or transcript to gauge compliance. Phrased as questions to ask or patterns to look for, not a restatement of the principle. Three rules for writing one:

- **Examples, not an enumeration.** The section is a starting point for judgment, not an exhaustive test suite. It should save the reader from re-deriving a check from first principles — it should not attempt to cover every case.
- **Name a language explicitly when a check would otherwise be too vague to act on.** This is a studio-level corpus with no fixed language or engine. "Check for null-check patterns" is too abstract to run; "in a language with a distinct `null`/`nil`, grep new code for `if x == null` / `if not x` guards and classify each" is concrete enough to actually do, even though the exact syntax differs per adopting project. Name the illustrative language rather than writing pseudo-universal syntax that fits none.
- **If no reasonable validation exists, say so plainly.** Some ADRs describe a norm about conduct, not a property of a diff (ADR-0001 is the clearest case in this corpus). Forcing a check onto that produces a check nobody will use. Write "No reliable validation is available; compliance is a matter of conduct, not diff inspection" rather than inventing a weak one.

## Reasons

### Why a fixed tag vocabulary instead of free text

A closed vocabulary is what lets an index-reading step behave mechanically: "does this ADR's tag intersect the categories of artifact this task touches" is a simple lookup against a known list. Free-text tags would turn that back into a judgment call at exactly the point it needs to be cheap.

### Why category, not file type

This corpus is shared across every Punch Up Games project, and those projects don't share a language or engine. A tag like `*.gd` is meaningless outside the one project that uses it and would need rewriting per adopter. `Code files` means the same thing everywhere; each project's own gate is responsible for mapping its actual file types onto these five categories.

### Why Validation is examples, not a checklist

An exhaustive checklist for a principle-shaped ADR is either long enough to become its own volume problem, or short enough to be wrong for half the real cases. A few well-chosen examples teach the reader *the kind of question to ask*, which generalizes better than a list they'd pattern-match against without thinking.

## Consequences

### Must do

- New ADRs in this corpus include both an `Applies to:` tag and a `## Validation` section from the start.
- Existing ADRs are updated with both as they're touched — tracked as its own pass across the corpus, not required all-at-once.
- Validation examples that involve syntax name the illustrative language explicitly.
- An ADR's one-line summary in the index table is updated to carry the `Applies to:` tag alongside it, so a reader can filter without opening the body (see the index for the mechanics).

### Must not do

- No inventing a file-type or language-specific tag value. If the fixed vocabulary doesn't fit an ADR, that's a signal to propose a new vocabulary entry here, not to freelance one on the ADR itself.
- No padding a Validation section to look complete. An honest "no reliable validation available" beats a token check nobody will use.
- Adding these two sections to an existing ADR must not change its Decision or Consequences content. This is a formatting pass, not a re-litigation.

## Validation

- Check any new ADR added to this corpus: does it have an `Applies to:` tag drawn only from the fixed vocabulary above, and a `## Validation` section (even if it states none is available)?
- Check any ADR edited under this convention: did the edit add only the header tag and the Validation section, or did it also change Decision/Consequences content along the way? The latter is a scope violation of this ADR's own "must not do."

## See Also

- ADR-0003 — Documentation Standards (this ADR is a documentation standard specific to the ADR corpus itself).
- The Studio ADR index (`000-index.md`) — consumes the `Applies to:` tag to decide which bodies a reader loads.
