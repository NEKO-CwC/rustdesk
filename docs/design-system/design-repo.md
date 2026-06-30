---
status: active
version: 0.1.0
last_reviewed: 2026-06-30
---

# RustDesk 二开设计 SSOT

## 范围

本设计 SSOT 是 RustDesk 二开仓库的最小有效设计入口。当前 feature 为核心网络能力，不改变 UI；因此本文件只固化已经存在的 Flutter 视觉和交互规则，供未来设置页或诊断页扩展引用。

## 证据入口

- Flutter 全局主题与 tokens：`flutter/lib/common.dart:250`。
- Flutter 亮色与暗色主题：`flutter/lib/common.dart:374`、`flutter/lib/common.dart:473`。
- 桌面按钮组件：`flutter/lib/desktop/widgets/button.dart:7`。
- 设置页 IPv6 P2P 开关：`flutter/lib/desktop/pages/desktop_setting_page.dart:574`、`flutter/lib/mobile/pages/settings_page.dart:800`。
- 弹窗与 overlay 管理：`flutter/lib/common.dart:788`、`flutter/lib/common.dart:1063`。

## LOCK 规则

- LOCK-1：UI-bearing work 必须使用 `MyTheme`、`Theme.of(context)`、`ColorThemeExtension` 或 `tokens/semantic.json` 中已有语义 token；不得在 feature 设计中重新定义品牌色或裸色值。
- LOCK-2：设置类二元项必须复用现有 switch / checkbox 模式；已有连接策略开关继续放在设置页网络区域。
- LOCK-3：按钮、弹窗、菜单和 overlay 必须复用已有 Flutter 组件或 `components/` 中的标准契约。
- LOCK-4：功能图标使用 Flutter `Icons` 或仓库 `IconFont`；不得用 emoji 作为功能图标。
- LOCK-5：所有 overlay 表面必须遵守 `foundations/z-index.md` 与 `components/overlay.md` 的层级、限高、内滚和 dismiss 可达规则。

## GUIDE 原则

- GUIDE-1：运维型设置界面保持紧凑、可扫描、少装饰。
- GUIDE-2：网络诊断信息优先面向开发和运维阅读，避免营销式文案。
- GUIDE-3：桌面端优先信息密度，移动端优先触控尺寸和换行。

## FREE 区域

- FREE-1：不改变 UI 的核心网络 feature 可以在 per-feature `design.md` 写 `Motion: N/A` 和 `Screenshot: N/A`。
- FREE-2：诊断日志只进入 Rust 日志时，不需要新增页面布局规则。

## Token 概览

- primitives: [tokens/primitives.json](tokens/primitives.json)
- semantic: [tokens/semantic.json](tokens/semantic.json)
- component: [tokens/component.json](tokens/component.json)
- token reference: [tokens/token-reference.md](tokens/token-reference.md)

## Focused Files

- Foundations: [color](foundations/color.md), [typography](foundations/typography.md), [spacing](foundations/spacing.md), [radius](foundations/radius.md), [elevation](foundations/elevation.md), [motion](foundations/motion.md), [iconography](foundations/iconography.md), [z-index](foundations/z-index.md), [breakpoints](foundations/breakpoints.md), [accessibility](foundations/accessibility.md)
- Components: [button](components/button.md), [dialog](components/dialog.md), [overlay](components/overlay.md), [settings-option](components/settings-option.md)
- Patterns: [interaction consistency](patterns/interaction-consistency.md)
- Content: [terminology](content/terminology.md)
- Conformance: [acceptance](conformance/acceptance.md)

## Agent 指令

每次 UI 相关工作先读本索引，再按受影响区域读取 focused files。per-feature `design.md` 只引用 `design_repo_refs`，不得复制或改写仓库级 token、motion 原语、组件规则。
