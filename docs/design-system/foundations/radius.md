# Radius

## 圆角尺度

| token | 值 | 用法 | 证据 |
| --- | --- | --- | --- |
| `radius.xs` | 2 | 小型装饰和细边界 | `flutter/lib/common.dart:645` |
| `radius.sm` | 5 | ListTile、桌面按钮 | `flutter/lib/common.dart:267`、`flutter/lib/desktop/widgets/button.dart:69` |
| `radius.md` | 8 | 弹窗、输入、ElevatedButton | `flutter/lib/common.dart:305`、`flutter/lib/common.dart:436` |
| `radius.pill` | 18 | TextButton | `flutter/lib/common.dart:387` |

## 规则

- LOCK：按钮、设置项和弹窗遵循现有圆角，不在 feature 层创建新的圆角家族。
- GUIDE：工具类 UI 的卡片和容器圆角不超过 `radius.md`。
