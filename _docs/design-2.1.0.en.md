# 2.1.0 Design Doc — Elegant Ink Light (Ink-Wash Rice-Paper Theme)

## Goal

Add an eye-friendly, ink-wash style light theme: the background simulates
pale-yellow rice paper (宣纸), with text and syntax highlighting rendered in
ink tones plus low-saturation traditional Chinese colors — the overall
impression of "mineral pigments painted on rice paper". Based on JetBrains'
official Islands Light theme style: colors only, no changes to layout or
component behavior.

## Structure

| File | Purpose |
|---|---|
| `resources/theme/ink-light.theme.json` | UI theme, `parentTheme: "Islands Light"` |
| `resources/theme/ink-light.theme.xml` | Editor scheme, `parent_scheme="Default"`, recolored key-for-key against `islands-light.theme.xml` |
| `plugin.xml` | Registers the `elegant-ink-light` themeProvider + `ElegantInkLight` colorScheme |

## Core mechanism: overriding the parent theme's semantic palette

The official Islands Light theme (`themes/islands/ManyIslandsLight.theme.json`
inside `intellij.platform.ide.impl.jar`) routes all of its UI mappings through
semantic token reference chains, e.g.:

```
ui:     MainWindow.background → main-window-bg → layer-0-bg → gray-150
```

When the platform loads a child theme (`importFromParentTheme()` in
`UIThemeBean.kt`), it merges both `colors` maps — **child entries win on name
collisions** — and the `ui` mappings inherited from the parent resolve against
the merged palette. So this theme's JSON only needs to override semantic
tokens (`layer-2-bg`, `accent-brand-bg`, …): hundreds of inherited ui mappings
turn warm automatically, while everything not overridden (gradients, badges,
component behavior) keeps the official defaults and tracks upstream Islands
updates for free.

**Important (do not delete)**: the tokens in `colors` are never referenced by
this file's own `ui` block — their consumers live in the parent theme. The
DevKit "unused color" inspection only scans this file and flags all of them as
unused; do not delete based on that inspection. The 66 tokens currently kept
were each verified to have a real consumer chain (direct parent-ui reference,
or transitive reference through other palette tokens). The following 9 tokens
were confirmed to have **no** consumer in Islands Light and were removed — do
not re-add them: `layer-0-bg-inline`, `layer-0-border-inline`,
`layer-1-border`, `control-bg-small-disabled`, `control-border-small`,
`toolbar-run-bg-hovered`, `toolbar-stop-bg-hovered`, `icon-green-stroke`,
`transparent`.

A few keys that bypass the palette as hardcoded hex in the parent
(`FileColor.*`, `Tag.background`, `Editor.ToolTip.selectionBackground`) are
overridden explicitly in the `ui` block. The `ToolWindow`/`Tree`/`Island` key
groups reuse the settings shared by this repo's Elegant Islands family.

The plugin manager page (Settings → Plugins) is another spot that bypasses the
semantic palette: the parent maps `Plugins.background` to the literal color
name `white` (#FFFFFF), while `Plugins.SectionHeader.background` is undefined
and falls through the wildcard `*.background` → `dialog-bg` → `layer-1-bg`
(cream in this theme), producing a patchwork of pure-white rows against cream
section headers. The `ui` block therefore overrides the `Plugins` key group
explicitly: `background` → `layer-2-bg` (paper), `SectionHeader.background` →
`layer-1-bg` (pinned explicitly instead of relying on the wildcard),
`SectionHeader.foreground` → `text-secondary`, `lightSelectionBackground` →
`selection-bg-active-muted`, and `tagBackground` → `#E4DBC2` (matching
`Tag.background`). Hover (`transparent-black-10`) and the search field
(`control-bg` → `layer-1-bg-inline`) inherit correctly from the parent and are
left untouched.

Editor file tabs, selected state (user-final): the focused window's active
tab fills with warm sand `layer-0-bg` (#EAE2CC, the window-base tone, set via
the explicit `EditorTabs.underlinedTabBackground` override in the `ui` block)
under the 黛青 `#44546B` underline; the active tab in an unfocused split goes
one paper step lighter — the `tab-selected-bg-inactive` token becomes
`#F1EAD6` (safe to change at token level: EditorTabs is its only consumer) —
with the `#CFC6AC` border kept.

Selection history, do not re-litigate: the initial `#E5EAE5` (hue ~120°
gray-green) read grayish on warm paper; the "lifted paper" variant (#F9F4E6)
read hollow; pale 黛青 `#E2E7ED` still felt detached to the user; warm sand
is final. The `tab-selected-bg-active` token itself stays pale 黛青
`#E2E7ED` — **never make it paper-colored** — it still feeds
`SearchEverywhere.Tab.selectedBackground` and `TabbedPane.focusColor`, where
a paper tone would make the focus tint invisible; that is exactly why the
active tab goes through an explicit `ui` key instead of the token.

## Color system

### Rice-paper background layers (UI)

| Layer | Value | Used for |
|---|---|---|
| `layer-2-bg` paper surface | `#F6F0DF` | Editor island, tool-window islands, popups |
| `layer-1-bg` | `#F1EAD6` | Dialogs |
| `layer-1-bg-inline` | `#F9F4E6` | Text fields, controls |
| `layer-0-bg` window base | `#EAE2CC` | Main-window background between islands, toolbar, status bar |

### Ink text

| Role | Value |
|---|---|
| Body, dense ink (浓墨) | `#3B3A36` (no pure black anywhere) |
| Secondary, medium ink (中墨) | `#6B675E` |
| Muted / disabled | `#5A5548` / `#9B937B` |

### UI accent (user-confirmed: 黛青 option)

| Role | Value |
|---|---|
| Accent, 黛青 (indigo-slate) | `#44546B` (buttons, tab underline, active toolbar) |
| Selection, pale cyan wash | `#DDE3E4` (tree/list selection) |
| Links, 黛蓝 | `#3D6A78` |
| Error / warning / success | 朱砂 `#B3554A` / 秋香 `#886715` / 苔绿 `#557441` (all desaturated) |

### Editor syntax colors (final)

The v1 syntax colors sat too close to body text in lightness (e.g. keyword
`#44546B` vs body `#3B3A36`); real-world feedback was "too weak". v2 raised
saturation one notch across the board while keeping lightness roughly
unchanged (the change touched only `ink-light.theme.xml`; the UI-level 黛青
accent is untouched):

| Element | v1 (weak) | v2 final | Traditional color |
|---|---|---|---|
| Keyword | `#44546B` | `#33568C` | 靛青 indigo |
| String | `#5F7355` | `#4A7A3D` | 竹绿 bamboo green |
| Number | `#9A6B48` | `#A85F2E` | 赭石 ochre |
| Function | `#3D6A78` | `#2E7093` | 青花 porcelain blue |
| Constant/field | `#7A5A72` | `#8E4D82` | 紫棠 plum purple |
| Class/interface | `#47756E` | `#2E7D6E` | 青碧 teal jade |
| Annotation/metadata | `#8A7742` | `#96781C` | 秋香 golden khaki |
| Error | `#B3554A` | `#C13E2F` | 朱砂 vermilion |
| Comment (italic, unchanged) | `#8F8873` | same | 淡墨 pale ink |

Eye-comfort baseline: all syntax foregrounds sit at roughly 4:1–6.5:1 contrast
against the paper surface (body text ≈ 9:1) — clearly softer than the default
Light theme's harsh primaries, while keeping syntax distinguishable.

### Terminal / console colors

Root cause: the scheme originally defined no `CONSOLE_*` keys at all — the
console/terminal background does not follow the `TEXT` background but inherits
`parent_scheme="Default"`'s pure white, and the ANSI 16 colors were the classic
primaries designed for a white base — so opening the terminal produced a harsh
white pane. Fix: define the full console block explicitly (mirroring the
approach in `islands-dark.theme.xml`).

Background `CONSOLE_BACKGROUND_KEY` = `#F6F0DF` (user-final: same paper tone
as the editor — in the Islands layout the terminal is an island too, sharing
one sheet of "paper"; the alternatives "one step darker #F1EAD6" and "one step
lighter #F9F4E6" were not chosen). Base keys: normal/system output `#3B3A36` /
`#6B675E`, error output vermilion `#C13E2F`, user input bamboo green `#4A7A3D`
italic (JetBrains convention), `CONSOLE_RANGE_TO_EXECUTE` effect `#698851`.

All 16 ANSI colors derive from the theme's existing ink palette: the normal
set reuses syntax/semantic tokens directly, the bright set lightens within the
same hue (WCAG contrast against the paper `#F6F0DF` in parentheses):

| ANSI | normal | bright |
|---|---|---|
| black | ink `#3B3A36` (10.0) | gray ink `#6B675E` (5.0) |
| red | terracotta `#B3554A` (4.3) | vermilion `#C13E2F` (4.6) |
| green | moss `#557441` (4.7) | bamboo `#4A7A3D` (4.5) |
| yellow | golden khaki `#886715` (4.6) | gold `#96781C` (3.7) |
| blue | indigo `#33568C` (6.5) | bright indigo `#4268A6` (4.9) |
| magenta | plum `#8E4D82` (5.2) | bright plum `#A34E9B` (4.5) |
| cyan | teal jade `#2E7D6E` (4.3) | bright jade `#2B8170` (4.1) |
| white | pale ink `#A79F89` (2.3) | paper white `#F9F4E6` (1.0) |

Trade-off (do not re-litigate): on a light terminal, ANSI white/bright-white
must stay near the base color — TUI programs mostly use them as reverse-video/
background colors (ANSI background colors share the same 16-color table), and
a dark value would turn those cases muddy; Solarized Light maps its white pair
the same way. `#4268A6`, `#A34E9B`, and `#2B8170` are the only newly tuned hex
values in this section (the palette had no existing lighter step in those
hues); everything else comes from existing tokens.

### Editor scheme additions relative to islands-light.theme.xml

- `DEFAULT_BRACES/BRACKETS/PARENTHS/COMMA/SEMICOLON/DOT/OPERATION_SIGN/IDENTIFIER`
  unified to dense ink (otherwise they inherit pure black from Default and
  clash with body text)
- `ERRORS_ATTRIBUTES` error squiggle softened to vermilion
- `DIFF_INSERTED/DELETED/CONFLICT` given paper-harmonious backgrounds (the
  original file only defined MODIFIED)
- `CARET_COLOR` dense ink
- The full `CONSOLE_*` block (background + base outputs + ANSI 16 colors, see
  "Terminal / console colors" above; islands-light is missing it too — the
  parent scheme Default's console is a pure-white base)
- Inlay hints (parameter/type hint chips — neither the official Light nor
  Islands scheme covers these, so they otherwise inherit Default's cool-gray
  chips `#EDEDED`/`#7A7A7A` plus the light-blue current parameter `#BCDAF7`):
  `INLINE_PARAMETER_HINT`/`INLAY_DEFAULT` bg `#EAE2CC` fg `#6B675E`
  (paper→chip luminance step mirrors the official white→#EDEDED, ~4.4:1),
  `INLINE_PARAMETER_HINT_HIGHLIGHTED` bg `#DDD4BB` fg `#5A5548`,
  `INLINE_PARAMETER_HINT_CURRENT` bg `#C7D2D6` fg `#44546B` (the 黛青 accent
  family, echoing the UI accent), `INLAY_TEXT_WITHOUT_BACKGROUND` fg `#7E7867`
  (slightly darker than the italic comment `#8F8873`, upright, so the two
  stay distinguishable)

## Maintenance guide

When tuning colors or investigating "this area is still cold gray":

1. Extract the official palette:
   `unzip -j "<IDE>/Contents/lib/intellij.platform.ide.impl.jar" "themes/islands/ManyIslandsLight.theme.json"`
2. Find the target area's key in its `ui` and see which token it references
   (mind token-to-token chains within the palette);
3. Override that token in `ink-light.theme.json`'s `colors` (preferred); only
   add an explicit `ui` key when the parent hardcodes a hex value;
4. Before adding a new token, verify its consumer chain: it must be referenced
   by the parent's ui directly, or transitively through other tokens that
   reach the ui — otherwise it is a dead variable.
