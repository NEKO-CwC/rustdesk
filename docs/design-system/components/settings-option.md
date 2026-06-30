# Component - Settings Option

## Metadata

- desktop evidence: `flutter/lib/desktop/pages/desktop_setting_page.dart:574`
- mobile evidence: `flutter/lib/mobile/pages/settings_page.dart:800`
- status: active

## Overview

设置项用于展示持久化配置，二元设置使用 checkbox 或 switch。

## Anatomy

- label
- optional leading icon
- control
- optional description or navigation affordance

## Tokens

- gap: `settingsOption.gap`
- label font: `settingsOption.labelFont`

## Props / API

- option key
- translated label
- current value
- onToggle / onChanged
- platform visibility guard

## States

- enabled
- disabled by policy
- toggled on
- toggled off
- incomingOnly / outgoingOnly hidden

## Do / Don't

- Do：网络连接策略继续放在设置页网络相关区域。
- Do：已有 option key 继续复用 `flutter/lib/consts.dart` 常量。
- Don't：为同一底层 option 创建第二个可见开关。

## Cross-refs

- `patterns/interaction-consistency.md`
- `foundations/accessibility.md`
