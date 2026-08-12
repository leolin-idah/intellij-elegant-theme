# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

An IntelliJ Platform UI theme plugin ("Elegant Theme") for JetBrains IDEs. There is no source code — the plugin is pure resources (theme JSON + editor color scheme XML) packaged into a jar.

## Building

There is no Gradle/CLI build. The project is an IntelliJ DevKit plugin module (`elegant-theme.iml`, type `PLUGIN_MODULE`):

- Build in IntelliJ IDEA via **Build → Prepare Plugin Module 'elegant-theme' For Deployment**, which produces `elegant-theme.jar` in the project root.
- Test changes by installing the jar via **Settings → Plugins → Install Plugin from Disk**, or by running the plugin in a DevKit sandbox IDE.
- `out`, `*.jar`, and `test` are gitignored; `out/production` is stale compiler output — edit files under `resources` only.

## Architecture

Everything the plugin ships lives in `resources`:

- `resources/META-INF/plugin.xml` — plugin descriptor. Each theme is registered as a **pair** of extensions: a `themeProvider` (UI theme, `.theme.json`) and a matching `colorScheme` (editor scheme, `.theme.xml`). The plugin version lives in the `<version>` tag here (nowhere else).
- `resources/theme/*.theme.json` — UI themes. Each defines a named color palette under `colors` and maps IntelliJ UI keys under `ui` (keys can reference palette names). Each points to its editor scheme via `editorScheme`.
- `resources/theme/*.theme.xml` (and `.icls`) — editor color schemes (`parent_scheme` is `Darcula` or `Light`).

Registered themes (in `plugin.xml`): Elegant Islands Light, Elegant Ink Light, Elegant Islands Dark, Elegant Dark. The Islands variants (including Ink Light) set `parentTheme` to JetBrains' Islands themes and layer adjustments on top. Note that `elegant-light.theme.json` exists in `resources/theme` but is **not** registered in `plugin.xml`.

`color-schema.json` at the repo root is a reference palette document, not consumed by the plugin.

## Conventions

- Releases: bump `<version>` in `plugin.xml` and add an entry to `CHANGELOG.md`. Commit messages historically just say "see CHANGELOG.md for details" — the changelog carries the real notes, including old→new hex values for color changes.
- When changing a color, keep the palette-name indirection: change the value in the theme JSON's `colors` block (or the scheme XML) rather than hardcoding hex values into individual `ui` keys.
- Design docs: every release that changes theme design gets a bilingual pair of design documents — `_docs/design-<version>.md` (Chinese) plus `_docs/design-<version>.en.md` (English) — recording design rationale, final palette, and maintenance notes. Keep the two in sync when either changes. See `_docs/design-2.1.0.md` / `_docs/design-2.1.0.en.md` (Elegant Ink Light) for the expected shape.
- Theme JSONs that set `parentTheme` may override the parent's semantic palette tokens in their `colors` block (e.g. `layer-2-bg`, `accent-brand-bg`). Such tokens are consumed by the parent theme's `ui` mappings at load time (`UIThemeBean.importFromParentTheme()` merges palettes, child wins), so the DevKit "unused color" inspection flags them falsely — do not delete them based on that inspection. See the maintenance guide in `_docs/design-2.1.0.md`.
