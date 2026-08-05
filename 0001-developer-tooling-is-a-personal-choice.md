# ADR-0001: Developer Tooling is a Personal Choice

**Status:** Accepted
**Date:** 2026-06-22
**Deciders:** Branden
**Applies to:** Always

## Context

This project's ADR set includes several records that govern working with AI coding tools — how to collaborate with them (ADR-0002), what they can do autonomously (ADR-0013), and how documentation is shaped for the contexts they create (ADR-0003). Those records are useful to contributors who use AI tools.

They should not imply that AI-assisted development is expected, required, or preferred.

Contributors come to a project with different tools, workflows, and considered stances. Some developers choose not to use AI tools for principled reasons — ethical, aesthetic, or practical. Some mix approaches depending on context. None of these stances is more or less legitimate than the others.

An ADR set that leads with AI collaboration norms without first establishing this risks reading as a de facto mandate. This ADR exists to make the project's actual position explicit before any other decisions are encountered.

## Decision

**No particular development tooling is assumed, required, or preferred at the project level.**

1. AI coding assistance — including agentic tools, LLM pair programming, and similar — is explicitly **optional**. It is one valid approach among several.
2. Contributors who use AI tools and contributors who do not are **equally valued**. Code and documentation are evaluated on their merits, not on how they were produced.
3. A contributor's tooling choices are **their own**. They are not subject to peer pressure, expectation of explanation, or judgment.
4. ADRs 0002, 0003, and 0013 govern how AI tools are used *when a contributor chooses to use them*. They create no obligations for contributors who don't.
5. This position holds regardless of organizational or business context. It is a professional value, not a governance requirement.

## Reasons

### Diverse working styles produce better outcomes

Teams that accommodate different approaches avoid monoculture. A developer who works without AI brings a different perspective on code clarity and legibility — one that has genuine value precisely because it doesn't share the assumptions that AI-assisted workflows can quietly embed.

### Developer autonomy over tooling is a foundational value

Telling a developer what tools to use — especially tools that touch questions of authorship, ethics, or data provenance — is a meaningful imposition. Where the project has no strict operational requirement (and this one doesn't), the choice belongs to the contributor.

### Explicit inclusion costs little and removes real friction

Making this decision explicit costs one document. Leaving it implicit costs contributors who read the AI-focused ADRs and wonder whether they're working the right way. The asymmetry justifies stating it plainly.

## Consequences

### For contributors who don't use AI tools

ADRs 0002, 0003, and 0013 do not apply to your workflow. All other ADRs apply in full. Code review, architectural expectations, and working norms are the same regardless of toolchain.

### For contributors who do use AI tools

ADRs 0002, 0003, and 0013 apply. The existence of this ADR doesn't change those commitments — it scopes them correctly.

### For code review and collaboration

Do not assume AI-generated content. Do not ask contributors to disclose whether they used AI tools unless there is a specific, stated reason the project needs to know. Evaluate output, not process.

### For future tooling decisions

New tooling decisions should state explicitly whether they create project-wide requirements or per-contributor defaults. Ambiguity recreates the problem this ADR was written to prevent.

## Validation

No reliable diff-level or transcript-level validation is available for this ADR. It governs how people talk about tooling choices — not a property of a diff or a plan. Compliance is a matter of conduct, checkable only by judgment:

- In a PR description, commit message, or review comment: does the language imply a tooling choice was required or expected ("should have used AI for this," "why wasn't this AI-assisted") rather than just describing what was done?
- In onboarding docs or a project's CLAUDE.md-equivalent: does it read as "AI assistance is available and optional," or does it read as an assumed default?

## See Also

- ADR-0002 — collaboration norms for AI-using contributors.
- ADR-0013 — AI permissions; operational scope of AI-assisted workflows.
- ADR-0016 — ADR authoring conventions (defines the `Applies to:` tag and `## Validation` section used here).
