# Component - Button

## Metadata

- canonical code: `flutter/lib/desktop/widgets/button.dart:7`
- related helper: `flutter/lib/common.dart:2949`
- status: active

## Overview

按钮用于明确命令。桌面端已有 `Button`、`FixedWidthButton` 和通用 `dialogButton()`。

## Anatomy

- 容器：`InkWell` + `Container`。
- 状态：hover、pressed、disabled。
- 文本：`translate()` 后渲染。

## Tokens

- background: `button.background`
- pressed: `button.pressedBackground`
- border: `button.hoverBorder`
- radius: `button.radius`
- font: `button.font`

## Props / API

- `text`
- `onTap` / `onPressed`
- `isOutline`
- `minWidth` / `width`
- `textSize`
- `radius`

## States

- default
- hover
- pressed
- disabled
- outline

## Do / Don't

- Do：清晰命令使用文本按钮或 icon + text。
- Do：长文本使用 `FixedWidthButton` 或能缩放/换行的容器。
- Don't：用普通 `Container` 自制按钮。

## Cross-refs

- `foundations/color.md`
- `foundations/radius.md`
- `foundations/typography.md`
