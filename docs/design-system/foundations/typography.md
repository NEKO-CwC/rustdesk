# Typography

## 字阶使用映射

| 角色 | token | Flutter 对应 | 密度规则 |
| --- | --- | --- | --- |
| 页面标题 | `font.input` | `Theme.of(context).textTheme.titleLarge` | 桌面和移动均保持系统字号，不在紧凑面板内放大 |
| 区块标题 | `font.body` | `TextStyle(fontSize: 14)` 或主题默认 | 设置页分组和弹窗正文标题 |
| 面板标题 | `font.body` | 主题默认 `Text` | 面板内标题不使用 hero 级字号 |
| 表头 | `font.caption` | `TextStyle(fontSize: 12)` | 表格和列表密集信息使用 caption |
| 正文 | `font.body` | Flutter 默认 body | 设置项说明、弹窗正文 |
| 标签 | `font.body` | `InputDecoration.labelText` | 输入框 label 和设置项 label |
| 辅助说明 | `font.caption` | `TextStyle(fontSize: 12)` | 校验错误、helper、状态说明 |
| 指标 | `font.body` | `TextStyle(fontSize: 14)` | 网络诊断数值保持可扫描 |
| 控件按钮 | `font.control` | `flutter/lib/desktop/widgets/button.dart:76` | 桌面按钮文字 |

## 证据

- 桌面按钮默认 `fontSize` 为 `12.0`，见 `flutter/lib/desktop/widgets/button.dart:76`。
- 设置相关 slider 数值和标签使用 `15` 左右的紧凑字号，见 `flutter/lib/common/widgets/setting_widgets.dart:72`。
- 弹窗错误文本使用 `fontSize: 12`，见 `flutter/lib/common/widgets/dialog.dart:447`。

## 规则

- LOCK：紧凑工具面板、设置页、诊断面板不得使用 hero 级大标题。
- LOCK：按钮和固定宽度按钮必须允许文本换行或缩放，避免长翻译溢出。
