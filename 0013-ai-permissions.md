# ADR-0013: AI Permissions — Intent and Best Practices

**Status:** Accepted
**Date:** 2026-05-22 (originally authored), adapted for general use
**Deciders:** Branden, Claude

**Scope:** This ADR applies to contributors using agentic AI coding tools. See ADR-0001.
**Applies to:** Always

## Context

The AI agent's harness (e.g. Claude Code) prompts the human before invoking certain tools — Bash commands, file writes outside the working tree, web requests, subagent spawns, and so on. The prompt is a safety mechanism. It's also a workflow interruption: every prompt is a context switch for the human, and many prompts ask about operations that are unambiguously fine inside this project's working agreement.

The goal is to set permissions so that **operations the project considers safe-by-default run without prompting, and operations that genuinely need a human check still surface.** Two failure modes to avoid:

1. **Over-permissive.** Allow-everything erases the value of the gate; surprising or destructive actions slip through.
2. **Over-restrictive.** Every step requires a prompt; flow shatters; humans start clicking "yes" without reading, defeating the gate.

A related concern: agents tend to "jazz" — improvise multi-step Bash pipelines instead of reaching for dedicated tools (Grep, Glob, Read, Edit) or established skills. Permissions are one lever to bias against this; norms are another.

## Decision

This ADR sets **project intent**. Each contributor implements it in their own settings file (project-local, not committed — see Consequences). The settings file *realizes* the intent and is allowed to evolve; the ADR is the contract the settings answer to.

### 1. Project file operations: unprompted

Reading, writing, editing, creating, and deleting files **inside the project tree** is freely permitted. This covers the `Read`, `Write`, `Edit`, `NotebookEdit` tools and also Bash-side equivalents (`rm`, `mv`, `mkdir`, etc.) when scoped within the workspace.

The AI is expected to use judgment on destructive ops (don't `rm -rf` a directory you haven't read), but the prompt machinery isn't the place where that judgment lives.

### 2. Source control: unprompted except the publish step

All version-control subcommands are unprompted **except the one operation with permanent external effect** — the publish/commit-to-shared-history step (e.g. `p4 submit`, `git push`), which requires explicit human approval.

Locally-destructive-but-recoverable operations (e.g. `p4 revert` / `p4 reconcile`, `git reset`) are reserved for the human's own workflow by **norm** rather than blocked by tooling — the AI does not initiate them. Creating changelists, drafting commit messages, opening files for edit/add/delete, staging, listing, diffing — all unprompted.

### 3. Jazz prevention: deny ad-hoc Bash for tasks dedicated tools handle

The AI has dedicated tools for reading (`Read`), searching contents (`Grep`), searching paths (`Glob`), editing (`Edit`), and waiting on processes (`Monitor`). Bash equivalents — `grep`, `find`, `cat`, `head`, `tail`, `sed`, `awk`, `echo` — are **denied by tooling** rather than discouraged by norm. The model already biases toward the right tool; the deny rule enforces that bias when bias slips.

When a recurring multi-step pattern doesn't fit a dedicated tool (e.g. "kick off the build and tail the log"), the response is to **wrap it as a skill or script**, not to thread a one-off pipeline through prompts. Skills are cheap to author and pay back across sessions; pipelines burn human attention every time.

If the AI hits a denied command and has no obvious tool substitute, the path is: tell the human, propose either a skill or a permission change, and proceed once the human picks.

### 4. Research tools (WebFetch, WebSearch): unprompted

External read-only research happens often and prompting on it interrupts flow. Both `WebFetch` and `WebSearch` are unprompted.

### 5. Subagent spawning (Agent, TaskCreate): prompted

Spawning a subagent burns context and tokens at a different scale than running a tool. Worth a heads-up. Stays prompted.

### 6. Anticipate-and-ask

When planning a task that will need permissions not currently granted, **surface this at planning time, not at execution time.** Propose the permission change with reasoning; let the human decide. The goal is "no surprise prompts mid-flow" — either the permission was already granted, or it was discussed and granted before the work started.

This applies symmetrically: if a permission is granted that's no longer needed, it's worth removing on the same beat.

## Reasons

### Why project intent in the ADR, implementation in user settings

The ADR is a contract about *what the project considers safe*. That contract should outlive any one contributor's machine setup. The settings file is *how this contributor wired their tooling to honor the contract*. Splitting these two means the contract is reviewable, debatable, and updatable independent of any one person's local configuration drift.

When best practice changes, the ADR changes first and the settings files follow. When a contributor wants to experiment locally (e.g. temporarily loosening a deny), they can do it without proposing a project-wide change.

### Why deny grep/find/cat/sed/awk/echo via tooling rather than norm

The model's bias toward dedicated tools is real but not absolute — under time pressure or in unfamiliar territory, it reaches for Bash one-liners. Norms are good at steady state; permissions are good at the moments when norms slip. Using both lets the norm carry the high-trust path (most of the time) and the permission catch the lapses.

Strict deny also has a side benefit: when the model *does* compose a Bash one-liner, the denial is informative — it surfaces the assumption that there's no better tool for the job, which is sometimes wrong (there usually is) and sometimes right (genuinely novel composition). The friction creates a checkpoint where a skill might get authored.

### Why gate the publish step but not local-recoverable operations

The publish step (`p4 submit`, `git push`) is the operation with permanent external effect — it puts work into shared history that others will pull. Gating it at the tool level matches the cost. Locally-destructive operations are recoverable through the human's normal workflow; they're reserved by norm rather than blocked by tooling because the human's preference there is workflow-shape, not safety. Different gates for different costs.

### Why subagent spawn is prompted

A subagent run can be 10–60 minutes of background compute and meaningful token cost. It's also opaque from outside — the human can't tell what the agent is doing without checking in. A prompt before spawn is a budget checkpoint, not a safety check.

## Consequences

### Settings file location and lifecycle

Each contributor maintains a project-local settings file (e.g. `.claude/settings.local.json`). This file is **not committed to source control** — it's per-contributor implementation of this ADR's intent.

The settings file should match the structure of the harness's official settings. When the ADR changes, contributors update their settings file. When a contributor wants a temporary deviation, it lives in their settings file and isn't argued at the project level until they decide it should be.

### Behavioral expectations on Claude

- Reach for the dedicated tool before composing Bash.
- When a recurring complex Bash pattern emerges, propose a skill.
- At planning time, scan for upcoming operations that aren't permitted; surface them and propose either a skill or a permission addition before starting.
- Never invoke the publish step (`p4 submit` / `git push`) without explicit approval, even if technically un-denied. Do not initiate local-destructive VCS operations (`revert` / `reconcile` / `reset`).
- When denied by tooling, treat that as signal — there's a better path; identify it before retrying.

### Behavioral expectations on the human

- When Claude proposes a permission change at planning time, decide it then, not under flow.
- When the prompt rate feels too high, mention it; the settings file is meant to evolve.
- When Claude jazzes through a problem that should have been a skill, call it out; the skill is the durable fix.

### Future work

- A starter settings file is checked into the contributor's personal setup (see the settings file authored alongside this ADR).
- A future ADR may codify "what makes something skill-worthy" if the threshold proves contentious in practice.

## Validation

- Check a session transcript for any publish-to-shared-history operation (language/tool-agnostic examples: `git push`, `p4 submit`) and confirm each was preceded by an explicit human approval *in that session* — not inferred from an earlier, unrelated approval.
- Check for "jazz": did the agent compose an ad-hoc multi-step shell pipeline for search-or-read work when a dedicated tool was available? Concretely, in a Claude Code session with Grep/Glob/Read tools available, a `grep`/`find`/`cat` Bash invocation for that purpose is exactly the pattern this ADR asks to be denied by tooling, not just discouraged by habit.
- At plan time: does the plan name any operation that will need a permission not currently granted, and propose it before execution starts — or does the first mention of it show up as a mid-execution prompt (the "anticipate-and-ask" failure this ADR calls out)?

## See Also

- ADR-0002 — collaboration norms; this ADR is the operational expression of "the human is final authority, the AI proposes."
- ADR-0003 — documentation standards.
- The project's rule file (e.g. `CLAUDE.md`) — operational rules; this ADR formalizes the permission-related ones.
- The contributor-local settings file (not committed) — the implementation file.
- ADR-0016 — ADR authoring conventions (defines the `Applies to:` tag and `## Validation` section used here).
