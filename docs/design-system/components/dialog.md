# Component - Dialog

## Metadata

- canonical code: `flutter/lib/common.dart:1063`
- manager: `flutter/lib/common.dart:788`
- status: active

## Overview

弹窗使用 `CustomAlertDialog` 和 `OverlayDialogManager`。设置、权限、连接失败、确认操作均复用该路径。

## Anatomy

- title
- content
- actions
- onSubmit
- onCancel
- overlay manager

## Tokens

- padding: `dialog.padding`
- radius: `dialog.radius`
- action spacing: `spacing.dialogPadding`

## Props / API

- `title`
- `content`
- `actions`
- `onSubmit`
- `onCancel`

## States

- default
- validation error
- loading
- destructive confirmation
- disabled action

## Do / Don't

- Do：设置修改和确认操作使用 `CustomAlertDialog`。
- Do：长内容弹窗内部滚动，关闭操作保持可达。
- Don't：在页面内自建固定定位 modal。

## Cross-refs

- `components/overlay.md`
- `foundations/z-index.md`
- `foundations/accessibility.md`
