# ADR-0015: The Three Virtues of a Programmer

**Status:** Accepted
**Date:** 2026-07-14
**Deciders:** Branden, Claude

## Context

> There are three great virtues of a programmer; Laziness, Impatience and Hubris
> 1. Laziness: The quality that makes you go to great effort to reduce overall energy expenditure. It makes you write labor-saving programs that other people will find useful and document what you wrote so you don't have to answer so many questions about it.
> 2. Impatience: The anger you feel when the computer is being lazy. This makes you write programs that don't just react to your needs, but actually anticipate them. Or at least pretend to.
> 3. Hubris: The quality that makes you write (and maintain) programs that other people won't want to say bad things about.  

This has a long lineage of being preserved:  
* Larry Wall, "Programming Perl", 2nd Edition, O'Reilly & Associates, 1996
* Reproduced online by Daniel Sherer
* Recovered by Wayback Machine and rehosted at [thethreevirtues.com](https://thethreevirtues.com/)

The framing is intentionally counterintuitive — each "virtue" sounds like a vice. That tension is the point: the qualities that look like flaws in a person (laziness, impatience, hubris) are, when applied with discipline to the *work*, the qualities that produce durable, useful software.

This ADR records the studio's adoption of the Three Virtues as a guiding framework, adds interpretation for how they shape day-to-day decisions (including AI contributor behavior), and links the canonical source for reference.

## Decision

The Three Virtues are adopted as a values framework for this studio. They sit alongside and are consistent with the norms in ADR-0002 (AI Collaboration Norms) — the Virtues describe how to approach the *work*; ADR-0002 describes how collaborators relate to *each other*.

### How this is interpreted into behavior

The virtues are not intuitive on first read. This section translates them into the concrete choices they govern.

#### Laziness — minimize total effort across the life of the project, not just today's effort

The lazy programmer is not the one who takes shortcuts. The lazy programmer is the one who invests just enough now to avoid paying many times over later.

- **"Laziest over time" is the standard, not "laziest right now."** A slightly harder choice today that makes every future decision cheaper is the lazier choice. A quick fix that creates five future headaches is a failure of Laziness.
- When facing a design choice, ask: *which option creates less total work across the project's lifetime?* The answer is usually the one with better naming, cleaner interfaces, and documentation that answers questions before they're asked.
- Prefer reusable solutions over one-offs when the cost of reuse is similar — the one-off will be reinvented.
- Document the *why*, not just the *what*. Answering a future question in a comment costs one unit of effort and saves N units across N future readers.
- Applied to implementation choices with uncertain scope: either do what's easiest now and design explicitly for replacement, or invest in the choice you'll wish you'd made three iterations later. Know which one you're doing and own it.

#### Impatience — anticipate needs; don't let avoidable friction stand

The impatient programmer doesn't accept slowness, repetition, or confusion as fixed conditions. They fix the structure.

- Anticipate the next question or need, not just the literal ask. If the answer to one question will immediately raise another, answer both.
- If you find yourself doing the same work twice, stop and build the abstraction. The third time you're copying something, it's already a component.
- Surface errors early and loudly. Silent failures and deferred problems become expensive problems. Make broken states impossible or unmissable.
- Slow builds, slow tests, and slow workflows are symptoms. Investigate and improve them; don't accept them as the cost of doing business.
- Write code that does the right thing by default, so callers don't have to carry implicit knowledge about what you wrote.

#### Hubris — take enough pride in the work to refuse to ship something embarrassing

The hubristic programmer writes code they can defend under scrutiny.

- Write code a skilled programmer would respect. If you'd be uncomfortable having a good engineer read it, that discomfort is information.
- "Temporary" and "quick fix" are not licenses for embarrassing code. The temporary almost always becomes permanent. Write it like it will last.
- Maintain quality standards in the parts no one sees. The invariant is the code, not the audience.
- Hubris does not mean perfectionism. A well-crafted solution that ships is better than a perfect solution that doesn't. The pride is in the *craft*, not in the delay.

## Reasons

The Three Virtues are specific where generic values are vague. "Write good code" produces no decisions. "Minimize total effort across the life of the project" produces specific ones. The virtues are also memorable precisely because they're counterintuitive — they stick.

Adopting them explicitly makes them available as shorthand in design conversations. When a collaborator (human or AI) is asked to choose between options, framing the question as "which is lazier over time?" activates a specific reasoning pattern rather than an appeal to taste.

## Consequences

### For all contributors (including AI)

- When evaluating implementation options, weight total future effort, not just immediate effort.
- When something is being done repeatedly, stop and remove the repetition.
- When something would be embarrassing if read by a skilled peer, don't ship it — or explicitly flag that you know it's a placeholder and why.
- Use the Virtues as shorthand in discussion: "the lazier choice here is X because..."

### For design discussions

The Virtues provide a common vocabulary for calls that would otherwise depend on taste:
- "Is this the laziest solution over time, or just the easiest right now?"
- "Are we being impatient enough — is this waiting for the user to know something it should anticipate?"
- "Would we be proud to have this read by someone skilled?"

## See Also

- [thethreevirtues.com](https://thethreevirtues.com/) — canonical source for the verbatim text quoted above
- Larry Wall, Tom Christiansen, Randal Schwartz, *Programming Perl*, 2nd Edition, O'Reilly & Associates, 1996 — original source
- ADR-0002 — AI Collaboration Norms (the Virtues govern the work; this ADR governs the relationship)
