# ADR-0014: Legacy Code and Transition Policy

**Status:** Accepted
**Date:** 2026-06-22
**Deciders:** Branden

## Context

These ADRs were adopted for a codebase that predates them. Most of the architecture decisions (ADRs 0004–0012) describe patterns the existing code does not yet follow.

Two bad responses to this situation:

1. **Mandate immediate compliance.** Rewrite everything to match the ADRs before touching anything else. This creates churn, blocks real work, and burns goodwill on abstract compliance.
2. **Treat the gap as permanent.** Accept that legacy code doesn't need to comply and stop expecting it to. This makes the ADRs aspirational-but-ignored — a credibility problem.

Neither is correct. This ADR defines the third path.

## Decision

### 1. ADRs are the north star for new code

All newly written code — new files, new systems, new features — follows the relevant ADRs from the start. No exceptions, no grace periods.

### 2. Legacy code is in transition, not in violation

Existing code that predates a relevant ADR is not treated as violating it. It is in transition. Transition is an explicit, acknowledged state — not a silent exception.

### 3. Maintenance does not increase non-compliance

When touching legacy code for any reason — bug fixes, small features, partial refactors — do not introduce patterns that move further from the relevant ADRs. Leave the code at least as compliant as you found it. Prefer the boy scout principle: leave it slightly more compliant if the opportunity is low-cost.

### 4. Refactors are opportunities, not obligations

When a piece of legacy code is formally refactored, the result should comply with the relevant ADRs. A refactor is the natural moment to bring it into alignment — the code is already being rewritten and the marginal cost is low.

Refactors are not required on a schedule. Compliance work is driven by genuine need — the code is being touched for another reason anyway.

### 5. Intentional exceptions are documented

If a piece of legacy code is expected to remain non-compliant indefinitely (e.g., a system too risky to refactor), document that exception in the relevant ADR or in a code comment with a brief explanation. Silence means "in transition," not "permanently exempt."

## Reasons

### Why "north star" rather than "mandatory target"

The ADRs describe the best way to write new code for this project. Forcing immediate full compliance on existing code produces surface-level compliance without the underlying design intentions — compliance theater that costs more than it's worth.

### Why not treat the gap as permanent

If legacy code is permanently exempt, the ADRs stop functioning as project norms and become stylistic preferences for some code but not others. New contributors can't reliably predict what they'll find. The ADRs lose their authority.

### Why maintenance shouldn't increase non-compliance

The slow drift toward worse patterns is more damaging than a single large violation. Each maintenance pass that adds non-compliant code shrinks the path to eventual compliance. The rule is asymmetric by design: compliance is encouraged, non-compliance is not extended.

## Consequences

### For new code

Follow the ADRs. If a relevant ADR seems wrong for the new code's situation, propose an amendment to the ADR rather than silently ignoring it.

### For legacy code

- In transition by default — no documentation needed unless the exception is intentional and indefinite.
- Maintenance passes don't add new violations.
- Refactors bring the code to compliance.

### For refactor planning

When scoping a refactor, compliance with relevant ADRs is a design goal of the work. The ADR index and individual ADRs may note which systems are in active transition.

### For code review

Reviewers check new code against ADRs. Reviewers do not require refactors of surrounding legacy code as a condition for merging a change, unless the change's scope already includes that code.

## See Also

- ADR-0004 through ADR-0012 — the architecture decisions this policy governs the adoption of.
