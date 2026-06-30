# Token Reference

## 来源

- `MyTheme` 定义 RustDesk 现有主题色、按钮色、边框色、圆角和控件主题，见 `flutter/lib/common.dart:250`。
- `ColorThemeExtension` 定义亮色与暗色扩展色，见 `flutter/lib/common.dart:144`。
- 桌面按钮直接消费 `MyTheme.accent`、`MyTheme.button`、`MyTheme.hoverBorder` 和 `MyTheme.border`，见 `flutter/lib/desktop/widgets/button.dart:55`。

## 使用规则

- LOCK：新 UI 只能引用 semantic 或 component token，或直接复用现有 Flutter 主题对象。
- LOCK：per-feature `design.md` 不写裸 hex、裸 px 或新字体定义。
- GUIDE：只有同一值在至少三个 UI 场景中稳定复用时，才晋升为新的 token。
