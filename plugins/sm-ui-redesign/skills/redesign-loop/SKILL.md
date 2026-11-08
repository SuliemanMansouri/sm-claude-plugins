---
name: redesign-loop
description: Iterate a redesign of one feature: designer canvas, blind UX and domain reviewers, an informed reviewer who triages against the decision log, repeated until criteria pass.
---

# Redesign loop

You are the **orchestrator** for one feature. You spawn every agent, you are the only writer of `loop.json`, and you never write canvases, reviews or triage yourself. Each agent writes exactly one file. Scope is **one feature and the views it opens** (tabs, dialogs, menus), never the whole app.

## Feature name

Every command takes a feature name, e.g. `/redesign 001-patient file redesign`, or just its number (`/redesign 001 next`).
- Folder: `docs/design/<slug>/`, where slug = the name lowercased, spaces and underscores turned into hyphens, other punctuation dropped: `001-patient-file-redesign`. Use the name as typed in titles and in the round summary.
- No number prefix given: assign the next free three-digit number from existing `docs/design/NNN-*` folders and tell the user.
- Existing feature: match by exact slug, then by number alone, then by name without the number. Several matches: ask. Never create a second folder with the same number.
- Read or create `docs/design/<slug>/feature.json` (`{number, name, slug, created}`). Store the slug in `loop.json` too.
- If the command omits the feature and exactly one loop has status running, use it; otherwise ask.
- `<feature>` below always means this slug.

## Commands

- `/redesign <feature>`: set up (first run) or show status.
- `/redesign <feature> next`: run one round.
- `/redesign <feature> override <round> <finding-id> accept|reject <reason>`: the user overrides triage.
- `/redesign <feature> stop`: end the loop and freeze the final canvas.

## Folder

```
docs/design/
├── DESIGN.md, css-vars.md          # from design-scan
└── <feature>/
    ├── feature.json                # number, name, slug, created
    ├── baseline/                   # from design-scan
    ├── brief.md                    # intent, users, tasks, constraints, sample cast
    ├── criteria.md                 # rubric, hard checks, stop rule
    ├── FLOW.md                     # decision log (informed reviewer maintains it)
    ├── loop.json                   # orchestrator state
    └── rounds/round-N/
        ├── canvas.html             # designer
        ├── canvas.aria.yml, shots/ # orchestrator runs the capture script
        ├── coverage.md             # baseline vs canvas control diff
        ├── review-ux.md            # blind
        ├── review-domain.md        # blind
        └── triage.md               # informed
```

Commit `docs/design/<feature>/` after every round, with the feature name and round in the message, so history lives in git, not only in chat or artifacts.

## Setup (first run)

1. If `docs/design/<feature>/baseline/` or `docs/design/DESIGN.md` is missing, run the `design-scan` skill first with the same feature name.
2. Write `brief.md` with the user: the problem in their words, intents (I1, I2... each one testable), users and roles, core tasks, domain constraints, locale, and a sample cast with realistic fictional data. Ask; do not invent domain rules.
3. Write `criteria.md`: a 1-5 rubric per intent plus usability, consistency with DESIGN.md, accessibility, locale; **hard checks** (every baseline control matched or removed on purpose; no colour outside css-vars; one primary action per screen if DESIGN.md says so; every screen has empty/error states); and the **stop rule**: no open blocker or major, every rubric item >= 4, and all hard checks passing in two consecutive rounds, or `max_rounds` (default 8).
4. Start `FLOW.md` titled with the feature name, with intents, vocabulary, and a Decisions section. Entry format:
   `D-012 (round 3) Decision. Why. Rejected: X (because ...). Source: finding UX-3.07 / user.`
5. `loop.json`: `{slug, name, round: 0, max_rounds, status, history: [{round, scores, blockers, majors, verdict}]}`.

## One round

1. **Designer** (fresh or resumed agent). Writes `round-N/canvas.html`.
2. **Capture** (you run it): open the canvas in Playwright, `ariaSnapshot()` each board into `canvas.aria.yml` with a heading per board, screenshot each board into `shots/`.
3. **Coverage diff** (you run it): extract `role "name"` pairs from every baseline file and from the canvas; match exact, then by role plus fuzzy name. Write `coverage.md` with Matched, Probably renamed, Removed, Added.
4. **Blind reviewers**, in parallel, **always freshly spawned** (never resumed, or they stop being blind): UX reviewer and domain reviewer.
5. **Informed reviewer**: triage, regressions, coverage, FLOW update.
6. **You**: update `loop.json`, apply the stop rule, publish the canvas as an artifact titled `<feature name> — round N` if the session can, and show the user a short round summary: scores, accepted/rejected counts, removals needing a decision. The user can override any triage line before the next round.
7. If a round's rubric total drops below the previous round, the informed reviewer must say whether to roll back; if it says roll back, the next designer starts from the better canvas.

## Agent briefs

Pass each agent the feature name and the paths below, nothing else. Every brief ends with: "Write exactly one file: <path>. Do not edit any other file."

**Designer**
- Reads: brief.md, criteria.md, DESIGN.md, css-vars.md, FLOW.md, baseline/, the previous canvas, the previous triage.md (accepted items only).
- Produces one self-contained `canvas.html`: one board per screen, tab, dialog and important state, each board a `<section aria-label="<Board name>">` with real semantic controls (buttons, inputs, tables) so the ARIA capture is meaningful. Colours only through the CSS variables in css-vars.md. Sample data from the brief's cast. Both locales if the app has two.
- Applies every accepted finding; never re-introduces anything FLOW.md rejected; keeps every baseline control unless FLOW.md or triage removed it on purpose.
- Ends the canvas with a hidden `<section aria-label="Changelog">` listing what changed this round and which finding each change answers.

**UX reviewer (blind)**
- Reads only: canvas.html, shots/, canvas.aria.yml, brief.md, criteria.md, DESIGN.md. **Not** FLOW.md, earlier rounds or the baseline.
- Judges usability heuristics, hierarchy, consistency with DESIGN.md, accessibility, states, locale/RTL.

**Domain reviewer (blind)**
- Reads only: canvas.html, shots/, brief.md, criteria.md.
- Plays the users in brief.md doing each core task; judges vocabulary, workflow fit, safety and missing domain steps. Ignores visual polish.

Both blind reviewers write this format:

```
# Review <ux|domain>: <feature name>, round N
Scores: I1 4, I2 3, Usability 4, ...
## Findings
- UX-N.01 [blocker|major|minor|nit] <board>: <problem>. Criterion: <id>. Suggest: <change>.
```

Severity rules: blocker = a core task cannot be done or is unsafe; major = a criterion fails; minor = it works but is worse than it should be; nit = taste. Every finding cites a criterion; findings without one are nits.

**Informed reviewer**
- Reads everything: all rounds, FLOW.md, baseline/, coverage.md, both reviews, the user's overrides.
- Writes `triage.md`:
  - Each blind finding: **accept** (designer must apply), **reject** (cite the FLOW decision or reason), **defer**, or **ask user** (domain rules only the owner can decide).
  - Regressions: anything that got worse since the previous round, or a FLOW decision the canvas now breaks.
  - Coverage: every Removed or Probably renamed control from coverage.md judged as intended (cite decision) or a mistake (becomes a major).
  - Hard-check results, rubric totals, and a verdict: CONTINUE, STOP (criteria met) or ROLL BACK.
  - FLOW changes, which it also applies to FLOW.md (the one exception to one-file-per-agent): new decisions with why and rejected options; duplicates merged; contradictions resolved or raised to the user.
- Blockers and majors that are accepted are mandatory next round; minors are optional; nits are logged only.

## Rules

- Never let a blind reviewer see history. Never let the designer skip triage.
- "Ask user" items stop the loop until answered; ask them in one AskUserQuestion batch.
- Never regenerate the baseline to make coverage pass; the baseline changes only through `design-scan`.
- When the loop stops, run the `redesign-handoff` skill with the same feature name if the user wants to build it.
