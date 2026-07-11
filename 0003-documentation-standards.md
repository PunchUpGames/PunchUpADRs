# ADR-0003: Documentation Standards

**Status:** Accepted
**Date:** 2026-05-20 (originally authored), adapted for general use
**Deciders:** Branden, Claude

## Context

This project is built collaboratively with an AI agent. Most project files will be read by the AI — repeatedly, across sessions — and sometimes by humans directly. Some documents are generated or maintained by AI; others are authored by hand with deliberate care.

Three levers govern documentation quality: locality (where it lives), concision (how much it says), and structure (what shape it takes for the reader). One additional concern: protecting human-authored documents from unintended AI modification.

## Decision

### 1. Human readability is the baseline

All documentation should be readable by a human who encounters it cold. This is the minimum bar, not a special category.

For documents where direct human reading is likely — design notes, ADRs, READMEs, planning docs — apply the **inverted pyramid**: most important information first, progressively more detail further down. A reader who stops at the first paragraph should still walk away with the most important thing.

This standard is strict. If a concept resists the pyramid shape, the concept isn't clear enough yet.

### 2. Protection flag for human-authored documents

Some documents reflect deliberate human authorship — chosen words, a considered voice, decisions made by a person. AI should not modify these without explicit instruction.

Mark human-authored documents with a callout near the top:

```
> **Human-authored** — Do not modify without explicit instruction.
```

Absence of this flag means AI may assist with generation, updates, and restructuring, subject to the other rules in this ADR. The flag is a write-permission signal, not an audience signal — both kinds of documents are written for humans to read.

### 3. Locality — documentation lives near what it describes

Whether AI-generated or human-authored, **documentation lives near the thing it documents.**

- Doc explaining a module sits in or next to that module
- Doc explaining a decision sits in the ADR index
- Doc explaining a plan sits next to the plan it's about
- File and directory structure encodes meaning — folder names announce what's inside, file naming carries scope information

Locality reduces context cost for the AI (less to load per query) and cognitive cost for the human (less to navigate).

### 4. Concise by default

Conciseness costs more to author and less to read. Lifetime cost — compute, context, attention — favors the shorter form. *"I would have written a shorter letter, but I did not have the time"* is the operating principle.

When a document grows beyond what its job requires, **extract subordinate details to their own document** rather than expanding the original. The primary doc holds what's load-bearing; the secondary doc holds the depth.

### 5. Code comments

Comments default to locality (live with the code they describe) and concision. Comments explain *why*, not *what* — the non-obvious reason a piece of code is the way it is. If a comment restates the code or the function name, delete it.

## Reasons

### Why human readability as the baseline, not a special category

The previous framing ("human-first vs. AI-first") implied that human readability was an exception requiring explicit marking. This has the wrong default: it produces documentation that serves AI well and humans poorly, with humans only getting consideration when someone remembered to ask for it. Most project documentation will eventually be read by a human — a new contributor, a code reviewer, a future collaborator. The default should serve them.

### Why a protection flag rather than an audience classification

Classifying documents by audience asks "who reads this?" — but the answer is usually "both, at different times." The useful question is "who may write this?" — specifically, whether AI may modify the document without instruction. The flag answers that directly. It also removes the implication that some documents are not for humans.

### Why inverted pyramid for human reading

Attention is the constraint. Readers stop; they scan; they read first paragraphs and skip the rest. The pyramid rewards attention as early as it arrives. Applied strictly, it also forces clarity: if the most important thing can't be stated first, the writer doesn't yet understand what the most important thing is.

### Why locality for both AI and human navigation

Context windows are finite and reloaded per session. Locality means the AI finds what it needs without loading the whole project. The same principle helps humans: structure-as-index is faster than text-as-index.

### Why concision is a discipline, not an instinct

Both default writers — models trained on internet text, and humans writing under time pressure — produce longer-than-necessary output. Conciseness must be applied as a deliberate pass.

## Consequences

### For all documentation

- Write for a human reader. Apply inverted pyramid where direct human reading is expected.
- Place the doc near what it documents.
- Be concise. If details are needed, extract them to a subordinate document.

### For human-authored documents

- Add the protection flag near the top.
- AI may not modify these without explicit instruction — including restructuring, shortening, or prose edits.

### For AI-generated or AI-maintained documents

- No flag needed. AI may generate, update, and restructure freely.
- Human readability still applies — even AI-maintained documents should be readable cold.

### Updating an existing doc

- Audit length before adding content. If the doc has grown past its purpose, extract or trim first.
- Audit location. If the doc has drifted from what it documents, move it.

### Code comments

- Default to no comment.
- Write a comment when the *why* is non-obvious — a hidden constraint, a workaround, a surprising invariant.
- Never restate what the code says.

## See Also

- [Inverted pyramid (Wikipedia)](https://en.wikipedia.org/wiki/Inverted_pyramid_(journalism))
- ADR-0002 — collaboration norms; establishes the AI working relationship that makes the protection flag meaningful.
