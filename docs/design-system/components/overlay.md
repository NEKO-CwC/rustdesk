# Component - Overlay

## Metadata

- manager: `flutter/lib/common.dart:788`
- mobile overlay evidence: `flutter/lib/mobile/pages/remote_page.dart:500`
- menu evidence: `flutter/lib/mobile/pages/remote_page.dart:792`
- status: active

## Overview

Overlay 家族包括 dialog、confirm、dropdown、select panel、field popover、combobox、menu、drawer、sheet、tooltip、toast。

## Anatomy

- trigger
- positioned surface
- scrollable content region
- dismiss affordance
- layer assignment

## Tokens

- layer: `foundations/z-index.md`
- radius: `radius.overlay`
- motion: `MOTION-M3`

## Props / API

- trigger context
- content builder
- close callback
- selected value callback

## States

- opening
- open
- scrolling
- dismissed
- blocked by validation

## Do / Don't

- Do：限高并让内容区域内部滚动。
- Do：保证 close / cancel / outside dismiss 可达。
- Do：dropdown 和 popover 高于 sticky header / sticky column。
- Don't：新增页面局部 z-index 数字体系。

## Cross-refs

- `foundations/z-index.md`
- `foundations/motion.md`
- `conformance/acceptance.md`
