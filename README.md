# sm-claude-plugins

Sulieman Mansouri's Claude skills and plugins, published as a Claude Code plugin marketplace.

## Install

```
/plugin marketplace add SuliemanMansouri/sm-claude-plugins
/plugin install sm-ui-redesign@sm-claude-plugins
```

For a local checkout: `/plugin marketplace add C:\Users\Sulieman\source\repos\sm-claude-plugins`.

## Plugins

### sm-ui-redesign

Iterative redesign of one feature at a time. All three skills take a feature name such as `001-patient file redesign` (stored as `docs/design/001-patient-file-redesign/`) or just its number.

| Skill | What it does | Writes |
|---|---|---|
| `design-scan` | Crawls one feature of the running app (pages, tabs, dialogs, menus), maps rendered values to the theme, asks about unclear usage | `docs/design/DESIGN.md`, `css-vars.md`, `<feature>/baseline/`, `drift.md` |
| `redesign-loop` | Rounds of designer canvas, blind UX and domain reviewers, and an informed reviewer who triages against the decision log, until the stop rule passes | `<feature>/brief.md`, `criteria.md`, `FLOW.md`, `rounds/round-N/` |
| `redesign-handoff` | Maps the final canvas to existing code as Reuse / Change / New, lists rule conflicts to confirm, slices the work | `<feature>/PLAN.md` |

Flow: `design-scan` → `redesign-loop` → `redesign-handoff` → build (e.g. with the factory plugin) → `design-scan` again for the new baseline.

## Layout

```
.claude-plugin/marketplace.json
plugins/<plugin>/.claude-plugin/plugin.json
plugins/<plugin>/skills/<skill>/SKILL.md
```

To add a plugin: create `plugins/<name>/` with its `plugin.json` and skills, then list it in `marketplace.json`.

## License

MIT © 2026 Sulieman Mansouri. Free to use, change and share; keep the copyright notice and credit. See [LICENSE](LICENSE).
