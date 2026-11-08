---
name: design-scan
description: Scan one feature of a running app (pages, tabs, dialogs, menus) to write DESIGN.md, an ARIA structure baseline and a drift list, asking the user where usage is unclear.
---

# Design scan

Turn an existing feature into written design rules and a structure baseline that later agents can read instead of looking at pictures. Works on any stack. First step of a redesign; also useful on its own.

## Feature name

Every run takes a feature name, e.g. `001-patient file redesign`.
- Folder: `docs/design/<slug>/`, where slug = the name lowercased, spaces and underscores turned into hyphens, other punctuation dropped: `001-patient-file-redesign`. Use the name as typed in titles.
- No number prefix given: assign the next free three-digit number from existing `docs/design/NNN-*` folders and tell the user.
- Existing feature: match by exact slug, then by number alone (`001`), then by name without the number. Several matches: ask. Never create a second folder with the same number.
- Record `{number, name, slug, created}` in `docs/design/<slug>/feature.json` if it does not exist yet; later skills read it.
- `<feature>` below always means this slug.

## Outputs

```
docs/design/
├── DESIGN.md                 # project-wide visual language (created or updated)
├── css-vars.md               # theme value -> CSS variable map for HTML canvases
└── <feature>/
    ├── feature.json          # number, name, slug, created
    ├── scan.md               # screen inventory, evidence tables, open questions
    ├── drift.md              # places where the feature breaks its own rules
    └── baseline/
        ├── README.md         # commit, date, account, data set, known limits
        ├── screens/*.aria.yml  # one Playwright ARIA snapshot per screen/dialog/menu/state
        └── shots/*.png       # one screenshot per snapshot
```

If `docs/design/DESIGN.md` already exists, run in **update mode**: keep every rule marked confirmed, add new evidence, and ask only about rules that are new or now contradicted.

## Inputs (ask in one batch if missing)

1. **Feature name** (see Feature name) and **entry route(s)**.
2. **Favourite screens** (1-2): tie-breakers when the feature disagrees with itself. Optional.
3. **How to run the app and sign in**: read CLAUDE.md, README, package scripts, launch settings first; ask only for what they do not answer. Prefer a demo or seeded data set; never use real patient/customer data.
4. Screens or areas to **exclude**.

## Step 1: detect the stack and the theme source

Read package.json / *.csproj / pyproject / config files. Find where design values live:

| Library | Theme source | Bypass to search for |
|---|---|---|
| MudBlazor | a `MudTheme` (PaletteLight/Dark, Typography, LayoutProperties) | `Style="..."`, hex in .razor/.css, `Class="pa-7"`-style one-offs |
| MUI | `createTheme(...)` | `sx={{ color: '#...' }}`, inline styles |
| Tailwind | tailwind.config / `@theme` | arbitrary values `text-[#..]`, `p-[13px]` |
| Vuetify / Angular Material / Bootstrap | theme config / SCSS variables | hex and px in component styles |
| Plain CSS | `:root` custom properties | hex, rgb, px outside variables |

Unknown library: treat CSS custom properties plus the most-used values as the theme.

## Step 2: scan the code of the feature

Locate the feature's files (routes, pages, dialogs, shared components it uses). Record with counts:
- Theme values (palette, type scale, radius, spacing, breakpoints).
- **Bypass values**: hardcoded colours, inline styles, magic pixel values, with file:line.
- Components and variants used, and how often (e.g. filled buttons per page, dialog vs drawer, chip vs badge, table vs list, icon set).
- Localization and RTL setup; copy patterns (sentence vs title case, verbs on buttons).

## Step 3: crawl the running feature

Start the app with the project's commands. With Playwright (`require('playwright')`, `chromium.launch()`), walk breadth-first from each entry route:

- For each screen and tab: `await page.locator('body').ariaSnapshot()` -> `screens/<Screen>.aria.yml`; a screenshot; and a computed-style sample of headings, body text, buttons by variant, inputs, cards, chips (font-size, weight, colour, background, padding, radius, gap).
- Open every control that opens something (`aria-haspopup`, menus, "+" buttons, row actions, buttons that produce a `role=dialog`). Snapshot it as `dialog-<name>` or `menu-<name>`, then close it with Escape or Cancel.
- Capture states where reachable: empty, loaded, validation error, disabled, read-only, other roles if the app has them.
- **Never trigger writes**: do not click Save, Submit, Sign, Pay, Delete, Send, Confirm, or anything irreversible. Open and close only. If a state needs a write, list it under known limits.
- Capture a second locale (e.g. Arabic/RTL) too if the app has one.

Map rendered values back to theme names (`#578ad6` = Palette.Primary). Values with no theme match are **proposed tokens**.

## Step 4: turn evidence into candidate rules

Categories: colour roles, typography, buttons and action hierarchy, layout and spacing, density, components (which for what), states (empty, loading, error, disabled vs hidden), forms and validation, dialogs vs panels, iconography, copy and tone, RTL/locale.

Classify each candidate rule in `scan.md`:
- **Consistent**: holds on ~90%+ of screens -> write as an observed rule, list exceptions in drift.md.
- **Favourite-resolved**: the feature is mixed but the favourite screens agree -> observed rule, the rest is drift.
- **Mixed**: no clear winner -> question.
- **No evidence**: intent, tone, why -> question only if it matters for redesign.
- **Not covered**: pattern never seen in the feature (charts, import, print, destructive confirm...) -> list under "Not covered" and ask for an example screen or a stated rule.

## Step 5: ask the user

Rank questions by how many screens they affect. Ask at most ~15, in AskUserQuestion batches of up to 4. Every question carries its evidence and a recommended default first:

> Filled buttons: one per screen on 31 of 39 screens; 2-3 on 8 (Visit, Record-labs, ...). Rule?
> 1. One filled button per screen; the 8 are drift (Recommended)  2. Up to two  3. No rule

Never ask what the code or crawl already answers. If nobody answers (unattended), write the default and mark it **unconfirmed**.

## Step 6: write the files

`DESIGN.md` template (keep it to 1-3 pages; rules, not essays):

```
# Design rules: <project>
Source: <feature name> scan, <date>, commit <sha>. Favourite screens: ...
Status key: [confirmed] user said so · [observed] evidence only · [unconfirmed] default, not yet answered

## Principles            (3-6 lines the user confirmed)
## Colour roles          (role -> theme token -> when to use)
## Typography            (scale step -> token -> use)
## Actions               (button hierarchy, primary count, destructive, disabled vs hidden)
## Layout and spacing    (grid, widths, spacing steps, density)
## Components            (pattern -> component + variant; e.g. "choice of one item: dialog, not drawer")
## States                (empty, loading, error, read-only, permission)
## Forms                 (labels, validation timing, required marks)
## Copy                  (case, verbs, tone, numbers/dates)
## Locale and RTL
## Not covered           (patterns with no rule yet)
```

Each rule names its framework mapping (e.g. "primary action -> `Variant.Filled Color.Primary`") and its status. `css-vars.md` lists every token with the CSS variable the framework emits (MudBlazor: `--mud-palette-*`) or one you define, so HTML canvases use the real palette. `drift.md` lists each violation with file:line or screen and the rule it breaks. `baseline/README.md` records the feature name, commit, date, account/role, data set and known limits, and states: "use this to check that a redesign keeps every control or removes it on purpose".

## Step 7: check before finishing

- Every rule has evidence counts or a user confirmation.
- Every screen in the inventory has a snapshot and a screenshot; every opened dialog/menu has one.
- No write action was triggered during the crawl.
- Tell the user: the feature folder, rules written (confirmed / observed / unconfirmed counts), drift items, not-covered patterns, and that `redesign-loop` can start from here with the same feature name.
