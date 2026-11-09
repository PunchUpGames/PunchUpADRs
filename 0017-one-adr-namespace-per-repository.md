# ADR-0017: One ADR Namespace per Repository

**Status:** Proposed
**Date:** 2026-09-24
**Deciders:** Branden, Claude
**Applies to:** Documentation, Plan files

## Context

Punch Up Games already runs several ADR corpora whose numbers overlap: this Studio corpus, each project's own, and shared infrastructure corpora. The index already asks readers to say "Studio ADR-000X" rather than a bare number. That works while there are only a few corpora. It stops working as shared repositories get mounted into more than one project, because a bare "ADR-0004" then has several plausible owners.

One shared corpus began life as a folder inside a single project's repository and is being extracted into its own repository so that several projects can share it. Whatever rule we adopt has to make that move cost nothing: its ADR numbers must survive the extraction.

## Decision

### 1. One namespace per repository

Each repository that holds ADRs numbers them independently of every other repository. Numbers are assigned in that repository and never change (see the Studio README).

### 2. Each namespace has a short registered name

A repository's ADR index declares one short name, unique across the adopting organisation. `Studio` is this repository, and is the only namespace this corpus names.

Each adopting organisation keeps a namespace registry outside this corpus, private if its project names are. This corpus never names a project. A repository's rule file (`CLAUDE.md` or equivalent) states where the registry is, unless the adoption note below covers it.

*Punch Up Games adoption note:* the registry is `https://github.com/PunchUpGames/studio-private-registry` (private). Other organisations: supply your own; this URL will not resolve for you.

A new repository's initial short name is chosen by a human and recorded in the registry before it is used. A project repository may say "Project ADR-000X" only inside its own documents, where the meaning is unambiguous.

### 3. Cross-repository references always carry the name

Outside the owning repository's own index, write `<Name> ADR-<number>`: "Studio ADR-0016", "Example ADR-0004". Never a bare number.

### 4. A repository that mounts another loads its index

When a repository mounts another's ADRs (for example as a git submodule), its rule file (e.g. `CLAUDE.md`) must list the mounted index and apply the same `Applies to` loading rule to it. Otherwise a contributor working in the mounting repository never sees decisions that bind it.

### 5. Namespaces are per repository, not per folder

A folder inside a repository does not get its own namespace. Scope within a repository is expressed with the `Applies to` tag and, where needed, a `Scope:` line on the ADR.

One exception, recorded so it isn't mistaken for a precedent: a folder that is **designated for extraction into its own repository** may carry a registered namespace while it still lives inside its host repository. Its numbers then move with it unchanged. Registering it early is what keeps the extraction from being a renumbering exercise.

### 6. Supersession across namespaces

When an ADR in one namespace supersedes one in another, the newer ADR cites the older by full name and number. The older ADR's status line is updated in the repository that still holds it. If the older ADR's repository no longer holds it, leave a short tombstone in the successor's namespace so the number isn't reused.

## Reasons

- **It matches how the work is already organized.** A repository has one owner, one review flow and one index. An ADR travels with the code it governs.
- **The extraction that motivated this stays a folder move.** Numbers are permanent identifiers; a rule that forced renumbering on extraction would break every existing citation.
- **Per-folder namespaces multiply cost without adding information.** Each needs its own index, its own qualifier in cross-references, and its own place in an agent's loading rules. The `Applies to` tag already answers "is this relevant to what I'm touching."
- **Registered names make the qualifier reliable.** Without a registry, two repositories can pick the same short name and recreate the ambiguity this ADR is meant to remove.

## Consequences

### Must do

- Declare a short name in each repository's ADR index and add it to the organisation's namespace registry (Decision §2).
- Use `<Name> ADR-<number>` in every cross-repository reference.
- List each mounted ADR index in the mounting repository's rule file.

### Must not do

- Don't reuse a registered name for a different repository.
- Don't name a project, or copy registry contents, into this corpus.
- Don't invent a namespace. If the registry can't be reached, stop and ask a human.
- Don't renumber an ADR when its corpus moves between repositories.
- Don't open a per-folder namespace except under the extraction exception in Decision §5.

## Validation

- A repository's rule file (`CLAUDE.md` or equivalent) states where the registry is, unless the adoption note in Decision §2 covers it. If the registry is unreachable, stop and ask a human rather than guessing.
- Check any new ADR index: does it declare a short name, and is that name in the organisation's registry and unused by another repository?
- Grep this corpus for project names or registry rows. Only `Studio` and placeholder names like `Example` should appear.
- Grep documents outside an ADR's own repository for a bare `ADR-` followed by digits with no name in front of it. In a Markdown repo: `grep -rnE '(^|[^A-Za-z-])ADR-[0-9]{4}' docs/` and inspect each hit that is not inside the owning corpus.
- For a repository that mounts another's ADRs (e.g. a `.gitmodules` entry pointing at an ADR corpus): does its rule file list that index?
- For any extraction of a folder into its own repository: do the ADR numbers before and after match one-to-one?

## See Also

- ADR-0016 — ADR authoring conventions (the `Applies to` tag this ADR relies on for scoping inside a repository).
- ADR-0003 — Documentation standards (locality: an ADR lives with what it governs).
- The Studio index (`000-index.md`) — points to where the namespace registry lives.
