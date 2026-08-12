# 2.1.0 设计文档 — Elegant Ink Light（水墨宣纸主题）

## 目标

新增一套护眼的水墨风浅色主题：背景模拟淡黄宣纸，文字与语法高亮用墨色 +
低饱和中国传统色，整体像「矿物颜料画在宣纸上」。基于 JetBrains 官方
Islands Light 新主题风格，只改颜色，不改任何布局与组件行为。

## 构成

| 文件 | 作用 |
|---|---|
| `resources/theme/ink-light.theme.json` | UI 主题，`parentTheme: "Islands Light"` |
| `resources/theme/ink-light.theme.xml` | 编辑器配色，`parent_scheme="Default"`，键位对照 `islands-light.theme.xml` 全量改色 |
| `plugin.xml` | 注册 `elegant-ink-light` themeProvider + `ElegantInkLight` colorScheme |

## 核心机制：覆盖父主题语义色板

官方 Islands Light（`intellij.platform.ide.impl.jar` 内的
`themes/islands/ManyIslandsLight.theme.json`）的所有 UI 映射都走语义 token
引用链，例如：

```
ui:     MainWindow.background → main-window-bg → layer-0-bg → gray-150
```

平台加载子主题时（源码 `UIThemeBean.kt` 的 `importFromParentTheme()`）会把
父子两边的 `colors` 合并、**同名键以子主题为准**，父主题继承下来的 ui
映射按合并后的色板解析。因此本主题的 JSON 只需覆盖语义 token
（`layer-2-bg`、`accent-brand-bg` 等），父主题几百条 ui 映射自动变暖，
渐变、徽章、组件行为等未覆盖项原样继承官方默认，官方升级 Islands 时可自动跟进。

**重要提示（防误删）**：`colors` 里的 token 不会被本文件的 `ui` 块引用，
消费者在父主题里。DevKit 的 unused 检查只扫本文件，会把它们全部误标为
未使用——不要据此删除。当前保留的 66 项全部经过引用链验证
（父主题 ui 直接引用 + 色板内传递引用 > 0）。以下 9 个 token 在 Islands
Light 中确认无消费者，已删除，勿加回：`layer-0-bg-inline`、
`layer-0-border-inline`、`layer-1-border`、`control-bg-small-disabled`、
`control-border-small`、`toolbar-run-bg-hovered`、`toolbar-stop-bg-hovered`、
`icon-green-stroke`、`transparent`。

个别绕过色板的硬编码键（`FileColor.*`、`Tag.background`、
`Editor.ToolTip.selectionBackground`）在 `ui` 块中显式覆盖；
`ToolWindow`/`Tree`/`Island` 三组键沿用本仓库 Elegant Islands 家族的既有设置。

插件管理页（Settings → Plugins）是另一处绕过语义色板的例外：父主题把
`Plugins.background` 直接映射到字面色名 `white`（#FFFFFF），而分组头
`Plugins.SectionHeader.background` 未定义、走通配符 `*.background` →
`dialog-bg` → `layer-1-bg`（本主题为米色），导致纯白列表行与米色分组头
拼接花斑。因此在 `ui` 块显式覆盖 `Plugins` 一组键：`background` →
`layer-2-bg`（纸面）、`SectionHeader.background` → `layer-1-bg`（显式固定，
不再依赖通配符）、`SectionHeader.foreground` → `text-secondary`、
`lightSelectionBackground` → `selection-bg-active-muted`、`tagBackground` →
`#E4DBC2`（与 `Tag.background` 一致）。悬停（`transparent-black-10`）与
搜索框（`control-bg` → `layer-1-bg-inline`）继承父主题即已正确，未覆盖。

编辑器文件 tab 的选中态（用户定稿）：聚焦窗口的 active tab 底色用
暖沙 `layer-0-bg`（#EAE2CC，同窗口底色，经 `ui` 块的
`EditorTabs.underlinedTabBackground` 显式覆盖）+ 黛青 `#44546B`
下划线；分屏失焦的 active tab 在此基础上浅一纸阶——
`tab-selected-bg-inactive` token 改为 `#F1EAD6`（该 token 只有
EditorTabs 一个消费者，改 token 安全），线维持 `#CFC6AC`。

选型过程记录，勿反复：初版 `#E5EAE5`（色相 ~120° 灰绿）在暖纸上
发灰；「亮纸提升」方案（#F9F4E6）显「镂空」；淡黛青 `#E2E7ED`
用户仍嫌浮，最终定暖沙。`tab-selected-bg-active` token 本身保持
淡黛青 `#E2E7ED` **不要改成纸色**——它还喂
`SearchEverywhere.Tab.selectedBackground` 和 `TabbedPane.focusColor`，
纸色会让 focus 色隐形，这正是 active tab 走 `ui` 显式键而非 token
的原因。

## 色彩系统

### 宣纸背景层次（UI）

| 层 | 色值 | 用途 |
|---|---|---|
| `layer-2-bg` 纸面 | `#F6F0DF` | 编辑器岛、工具窗口岛、弹窗 |
| `layer-1-bg` | `#F1EAD6` | 对话框 |
| `layer-1-bg-inline` | `#F9F4E6` | 输入框、控件 |
| `layer-0-bg` 窗口底 | `#EAE2CC` | 岛屿之间的主窗口底色、工具栏、状态栏 |

### 墨色文字

| 角色 | 色值 |
|---|---|
| 正文 浓墨 | `#3B3A36`（全局无纯黑） |
| 次级 中墨 | `#6B675E` |
| 弱化 | `#5A5548` / `#9B937B` |

### UI 强调色（用户确认：黛青方案）

| 角色 | 色值 |
|---|---|
| 强调 黛青 | `#44546B`（按钮、Tab 下划线、工具栏激活） |
| 选中 淡青晕 | `#DDE3E4`（树/列表选中） |
| 链接 黛蓝 | `#3D6A78` |
| 错误/警告/成功 | 朱砂 `#B3554A` / 秋香 `#886715` / 苔绿 `#557441`（均降饱和） |

### 编辑器语法色（最终版）

初版语法色与正文明度过近（如关键字 `#44546B` vs 正文 `#3B3A36`），实测
反馈「太弱」，v2 整体提一档饱和度、明度基本不动（调整仅涉及
`ink-light.theme.xml`，UI 层黛青不变）：

| 元素 | v1（弱） | v2 最终 | 传统色名 |
|---|---|---|---|
| 关键字 | `#44546B` | `#33568C` | 靛青 |
| 字符串 | `#5F7355` | `#4A7A3D` | 竹绿 |
| 数字 | `#9A6B48` | `#A85F2E` | 赭石 |
| 函数 | `#3D6A78` | `#2E7093` | 青花 |
| 常量/字段 | `#7A5A72` | `#8E4D82` | 紫棠 |
| 类/接口 | `#47756E` | `#2E7D6E` | 青碧 |
| 注解/元数据 | `#8A7742` | `#96781C` | 秋香 |
| 错误 | `#B3554A` | `#C13E2F` | 朱砂 |
| 注释（斜体，未调） | `#8F8873` | 同左 | 淡墨 |

护眼底线：所有语法前景色在纸面上的对比度约 4:1–6.5:1（正文约 9:1），
显著低于默认 Light 主题的刺眼原色，但保证语法区分度。

### 终端 / Console 配色

问题根源：scheme 原本一个 `CONSOLE_*` 键都没定义，console/terminal
背景不跟随 `TEXT` 背景，而是继承 `parent_scheme="Default"` 的纯白，
ANSI 16 色也全是为纯白底设计的经典原色——打开终端就是一块刺眼的白。
修复：显式定义整块 console 配色（对照 `islands-dark.theme.xml` 的做法）。

背景 `CONSOLE_BACKGROUND_KEY` = `#F6F0DF`（用户定稿：与编辑器同纸色，
Islands 布局里终端也是一个 island，共用一张「纸」；备选「深一档
#F1EAD6」「浅一档 #F9F4E6」未选）。基础键：普通/系统输出 `#3B3A36` /
`#6B675E`，错误输出朱砂 `#C13E2F`，用户输入竹绿 `#4A7A3D` 斜体
（JetBrains 惯例），`CONSOLE_RANGE_TO_EXECUTE` 效果色 `#698851`。

ANSI 16 色全部从主题既有墨水色系派生，normal 直接复用语法/语义
token，bright 同色相提亮（括号内为对纸面 `#F6F0DF` 的 WCAG 对比度）：

| ANSI | normal | bright |
|---|---|---|
| black | 墨 `#3B3A36` (10.0) | 灰墨 `#6B675E` (5.0) |
| red | 陶土 `#B3554A` (4.3) | 朱砂 `#C13E2F` (4.6) |
| green | 苔绿 `#557441` (4.7) | 竹绿 `#4A7A3D` (4.5) |
| yellow | 秋香 `#886715` (4.6) | 金黄 `#96781C` (3.7) |
| blue | 靛青 `#33568C` (6.5) | 亮靛 `#4268A6` (4.9) |
| magenta | 紫棠 `#8E4D82` (5.2) | 亮紫 `#A34E9B` (4.5) |
| cyan | 青碧 `#2E7D6E` (4.3) | 亮青 `#2B8170` (4.1) |
| white | 淡墨 `#A79F89` (2.3) | 纸白 `#F9F4E6` (1.0) |

取舍说明（勿反复）：浅色终端里 ANSI white/bright-white 只能取
「近底色」——它们更多被 TUI 程序用作反白/背景色（ANSI 背景色共用同
一张 16 色表），取深色会让反白场景变泥；Solarized Light 同样把
white 一对映射到近底色。`#4268A6`、`#A34E9B`、`#2B8170` 三个 bright
值是本节新调的 hex（色板中无现成同色相亮阶），其余全部来自既有 token。

### 编辑器 scheme 相比 islands-light.theme.xml 的补充键

- `DEFAULT_BRACES/BRACKETS/PARENTHS/COMMA/SEMICOLON/DOT/OPERATION_SIGN/IDENTIFIER`
  统一浓墨（否则继承 Default 的纯黑，与正文冲突）
- `ERRORS_ATTRIBUTES` 波浪线降为朱砂
- `DIFF_INSERTED/DELETED/CONFLICT` 补充纸面和谐色（原文件只有 MODIFIED）
- `CARET_COLOR` 浓墨
- `CONSOLE_*` 整块（背景 + 基础输出 + ANSI 16 色，见上节「终端 / Console
  配色」；islands-light 同样缺失，父 scheme Default 的 console 是纯白底）
- Inlay hints（参数名/类型提示芯片，官方 Light/Islands scheme 均未覆盖，
  否则继承 Default 的冷灰芯片 `#EDEDED`/`#7A7A7A` 加浅蓝当前参数
  `#BCDAF7`）：`INLINE_PARAMETER_HINT`/`INLAY_DEFAULT` 底 `#EAE2CC`
  字 `#6B675E`（纸→芯片明度差复刻官方 白→#EDEDED，约 4.4:1）、
  `INLINE_PARAMETER_HINT_HIGHLIGHTED` 底 `#DDD4BB` 字 `#5A5548`、
  `INLINE_PARAMETER_HINT_CURRENT` 底 `#C7D2D6` 字 `#44546B`（黛青系，
  与 UI 强调色呼应）、`INLAY_TEXT_WITHOUT_BACKGROUND` 字 `#7E7867`
  （比斜体注释 `#8F8873` 略深，直立体，两者可区分）

## 维护指南

后续调色 / 排查「某区域还是冷灰」时：

1. 解包官方色板：
   `unzip -j "<IDE>/Contents/lib/intellij.platform.ide.impl.jar" "themes/islands/ManyIslandsLight.theme.json"`
2. 在其 `ui` 中找到目标区域的键，看它引用哪个 token（注意 token 之间的链式引用）；
3. 在 `ink-light.theme.json` 的 `colors` 里覆盖该 token（优先），仅当父主题
   硬编码 hex 时才在 `ui` 块加显式键；
4. 新增 token 前先验证消费链：token 需被父主题 ui 直接引用，或被其他 token
   传递引用后进 ui，否则是死变量。
