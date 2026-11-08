---
name: redesign-handoff
description: Turn a finished redesign canvas and its decision log into a build plan: map every item to existing code as reuse, change or new, list rule conflicts, and slice the work.
---

# Redesign handoff

Bridge from a finished redesign (usually the last round of `redesign-loop`) to implementation. Output: `docs/design/<feature>/PLAN.md`. Nothing is built by this skill.

## Feature name

Takes a feature name, e.g. `001-patient file redesign`, or just its number (`001`).
- Folder: `docs/design/<slug>/`, where slug = the name lowercased, spaces and underscores turned into hyphens, other punctuation dropped: `001-patient-file-redesign`.
- Match by exact slug, then by number alone, then by name without the number. Several matches: ask. No match: say so and suggest `redesign-loop` or `design-scan` first; never create a new numbered folder here.
- Read `docs/design/<slug>/feature.json` for the display name; use it in the PLAN.md title.
- `<feature>` below always means this slug.

## Inputs

- Final `canvas.html` and `canvas.aria.yml` from the last round (or an artifact link the user gives).
- `FLOW.md`, `brief.md`, `DESIGN.md`, `baseline/`, and `coverage.md` of the last round.
- The codebase, plus project conventions (CLAUDE.md, architecture docs, project skills).

## Steps

1. **Inventory the design.** List every board, section, control and rule in FLOW.md as a design item. Group by screen and by domain area.
2. **Map to code.** For each item find the code that does it today, with file:line. Fan out with Explore agents per area when the feature is large; each returns a table, not file dumps.
3. **Bucket each item:**
   - **Reuse**: behaviour exists; only UI moves or is relabelled. Client change only.
   - **Change**: behaviour exists but a rule, field or timing changes. Needs domain, contract or data changes.
   - **New**: nothing exists. Full vertical slice per project conventions.
4. **Find conflicts**: every place where the design contradicts a rule the code enforces today (timing, permissions, billing, finality, validation). Table: id, conflict, today (with file:line), design default, affected slices. These are **decisions to confirm**; ask the user in AskUserQuestion batches, with the design's default first. Unanswered ones stay listed as open.
5. **Removed controls**: from coverage.md, confirm every removed baseline control has a FLOW decision; list the code that will be deleted.
6. **Slice and order.** Slices sized for one build job or PR, numbered under the feature (e.g. `001-2.3`). Start with a foundations phase (renames, shared types, guards); then a UI-only phase that ships the new layout on today's behaviour; then rule-change phases. Show dependencies as a small text diagram and mark what can run in parallel.
7. **Risks**: data migration and history/replay, offline or caching, permissions, the largest slices, items waiting on outside answers.
8. **Finish line**: re-run `design-scan` with the same feature name against the built feature to capture the new baseline; update DESIGN.md if the redesign changed visual rules; delete the old baseline once applied.

## PLAN.md layout

```
# <Feature name>: implementation plan
Status, date. Target design link. Decision log link. Baseline link. Vocabulary and path shorthand.
1. How to read this plan (bucket table, build rules)
2. Decisions to confirm (conflict table)
3. Mapping (per area: design item | bucket | today file:line | work)
4. Phases and slices (per slice: owner module, domain, events/contracts, API, data, tests)
5. Order and size (dependency diagram, slice counts)
6. Risks
```

## Check before finishing

- Every design item appears exactly once in the mapping.
- Every Change and New item belongs to a slice; every slice names its tests.
- Every file:line reference was opened, not guessed.
