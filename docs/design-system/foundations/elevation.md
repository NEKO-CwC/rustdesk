# Elevation

## 阴影与层次

| 用法 | 规则 | 证据 |
| --- | --- | --- |
| 普通按钮 | elevation 为 0 | `flutter/lib/common.dart:2970` |
| 弹窗 | 使用 Flutter `AlertDialog` 与主题，不自建阴影 | `flutter/lib/common.dart:1063` |
| toast / attached widget | 使用现有 BotToast 或 Overlay 管理 | `flutter/lib/desktop/widgets/tabbar_widget.dart:75` |

## 规则

- LOCK：新 overlay 不创建页面局部阴影系统。
- GUIDE：远程桌面工具面优先清晰边界和层级顺序，避免装饰性投影。
