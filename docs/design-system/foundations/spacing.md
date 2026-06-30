# Spacing

## 间距尺度

| token | 值 | 用法 |
| --- | --- | --- |
| `spacing.xs` | 4 | 图标与短文本、输入附属控件 |
| `spacing.sm` | 8 | 表单字段、弹窗局部间距 |
| `spacing.md` | 12 | 行内分组、按钮水平内距 |
| `spacing.lg` | 24 | 弹窗 padding 和大分组 |

## 证据

- 弹窗 padding 由 `MyTheme.dialogPadding = 24` 驱动，见 `flutter/lib/common.dart:317`。
- 桌面按钮默认 `padding` 为 `4.5`，见 `flutter/lib/desktop/widgets/button.dart:52`。
- 弹窗字段间隔常用 `SizedBox(height: 8.0)`，见 `flutter/lib/common/widgets/dialog.dart:205`。

## 规则

- LOCK：弹窗、按钮、设置项优先复用现有组件的内距。
- GUIDE：网络诊断信息如果以后进入 UI，应使用紧凑行距，便于比较日志状态。
