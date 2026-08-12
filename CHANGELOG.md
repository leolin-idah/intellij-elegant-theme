## 2.1.0

Add new theme: **Elegant Ink Light(Beta)** — an eye-friendly ink-wash (水墨) style light theme based on Islands Light.

- Rice-paper (宣纸) warm backgrounds: editor #F6F0DF, window #EAE2CC
- Ink-tone text: primary #3B3A36, secondary #6B675E, comments #8F8873 (italic)
- Traditional-color syntax accents: keyword 靛青 #33568C, string 竹绿 #4A7A3D, number 赭石 #A85F2E, function 青花 #2E7093, constant 紫棠 #8E4D82, class 青碧 #2E7D6E, error 朱砂 #C13E2F
- UI accent (buttons, tab underline, selection) uses 黛青 #44546B; full warm-tinted palette override of the Islands Light color ramps
- Plugin manager page follows the rice-paper palette: list/details background #FFFFFF → #F6F0DF, section header #F1EAD6, selected row #E5EAE5, tag #E4DBC2 (upstream maps `Plugins.background` to the literal `white`, bypassing semantic tokens)
- Inlay hints (parameter/type hints) recolored to warm ink chips: default #EDEDED/#7A7A7A → #EAE2CC/#6B675E, hover #DDD4BB/#5A5548, current parameter #BCDAF7 → #C7D2D6/#44546B (黛青), no-background inlay text #7E7867
- Editor file tabs: active tab fill is now warm sand #EAE2CC (window-base tone) under the 黛青 #44546B underline, replacing the gray-green #E5EAE5; the active tab in an unfocused split lightens one paper step to #F1EAD6. The shared `tab-selected-bg-active` token is re-tinted to pale 黛青 #E2E7ED for its remaining consumers (Search Everywhere selected tab, Settings `TabbedPane.focusColor`)
- Terminal / console recolored to the ink palette (the scheme previously defined no `CONSOLE_*` keys, so the terminal opened on Default's pure-white background): background #FFFFFF → #F6F0DF (paper), normal output #3B3A36, system output #6B675E, error output 朱砂 #C13E2F, user input 竹绿 #4A7A3D italic, plus a full 16-color ANSI ramp derived from the scheme's ink tones — black 墨 #3B3A36/#6B675E, red 朱砂 #B3554A/#C13E2F, green 竹绿 #557441/#4A7A3D, yellow 秋香 #886715/#96781C, blue 靛青 #33568C/#4268A6, magenta 紫棠 #8E4D82/#A34E9B, cyan 青碧 #2E7D6E/#2B8170, white #A79F89/#F9F4E6

## 2.0.1

Optimize the UI display of the settings panel.

## 2.0.0

Based on the solid color scheme of JetBrains' latest Islands theme, I have made a few adjustments specifically to optimize the Islands Dark experience.
