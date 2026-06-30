# Color

## 语义映射

| 语义角色 | token | 现有代码证据 | 用法 |
| --- | --- | --- | --- |
| 主操作色 | `color.action.primary` | `flutter/lib/common.dart:254` | 主要按钮、强调图标、连接动作 |
| 按钮底色 | `color.action.button` | `flutter/lib/common.dart:263` | 桌面按钮默认背景 |
| 默认边框 | `color.border.default` | `flutter/lib/common.dart:258` | 输入、按钮、分隔边界 |
| hover 边框 | `color.border.hover` | `flutter/lib/common.dart:264` | 桌面 hover 反馈 |
| 应用浅背景 | `color.surface.app` | `flutter/lib/common.dart:253` | 浅色背景区域 |
| 深色画布 | `color.surface.canvasDark` | `flutter/lib/common.dart:257` | 远程画布和深色承载面 |
| 弱文本 | `color.text.muted` | `flutter/lib/common.dart:260` | 次要说明、弱提示 |

## 规则

- LOCK：UI-bearing work 复用 `MyTheme`、`Theme.of(context)`、`ColorThemeExtension` 或 semantic token。
- LOCK：状态色必须来自 Flutter `ColorScheme` 或已有主题扩展；不得在 feature 文件内自定义品牌色。
