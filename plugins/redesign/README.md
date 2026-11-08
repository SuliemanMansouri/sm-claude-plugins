# redesign

Iterative redesign of one feature of an app at a time. The plugin turns an existing feature into written design rules and a structure baseline, runs rounds of design and review until agreed criteria pass, then hands off a build plan mapped to your existing code. It works on any stack.

All three skills take a feature name such as `001-patient file redesign` (stored as `docs/design/001-patient-file-redesign/`) or just its number (`001`).

## Skills

| Skill | What it does | Writes |
|---|---|---|
| `design-scan` | Crawls one feature of the running app (pages, tabs, dialogs, menus), maps rendered values to the theme, and asks you about unclear usage | `docs/design/DESIGN.md`, `css-vars.md`, `<feature>/baseline/`, `scan.md`, `drift.md` |
| `redesign-loop` | Runs rounds of a designer canvas, blind UX and domain reviewers, and an informed reviewer who triages findings against the decision log, until the stop rule passes | `<feature>/brief.md`, `criteria.md`, `FLOW.md`, `loop.json`, `rounds/round-N/` |
| `redesign-handoff` | Maps the final canvas to existing code as Reuse / Change / New, lists rule conflicts for you to confirm, and slices the work into ordered build steps | `<feature>/PLAN.md` |

Typical flow: `design-scan` → `redesign-loop` → `redesign-handoff` → build → `design-scan` again to capture the new baseline.

## What the plugin runs and touches

- **Your app, locally.** `design-scan` starts your app with the project's own commands and walks it with Playwright to take ARIA snapshots and screenshots. `redesign-loop` opens its HTML canvases in Playwright the same way. Playwright must already be available in your project; the plugin installs nothing.
- **Files in your repository.** Everything is written under `docs/design/`. `redesign-loop` commits that folder after each round.
- **Subagents.** `redesign-loop` and `redesign-handoff` start Claude subagents for designing, reviewing and code mapping.
- **Artifacts (optional).** If your Claude session can publish artifacts, `redesign-loop` publishes each round's canvas as a private artifact.

The plugin contains no scripts, hooks or MCP servers, and sends no data to any other service.

## Install

```
/plugin marketplace add SuliemanMansouri/sm-claude-plugins
/plugin install redesign@sm-claude-plugins
```

## License

MIT © 2026 Sulieman Mansouri.
