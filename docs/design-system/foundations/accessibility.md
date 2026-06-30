# Accessibility

## 基线

- 交互控件必须使用 Flutter 标准按钮、switch、checkbox、radio、menu 或可聚焦组件。
- 设置项文案必须经过 `translate()` 或现有本地化表。
- 弹窗必须提供可达的取消或关闭路径。

## 证据

- 设置页 IPv6 P2P 开关使用现有设置控件，见 `flutter/lib/desktop/pages/desktop_setting_page.dart:574`、`flutter/lib/mobile/pages/settings_page.dart:800`。
- 弹窗按钮通过 `dialogButton()` 构造，见 `flutter/lib/common.dart:2949`。
- 弹窗 `onCancel` 路径在多个设置弹窗中传入，见 `flutter/lib/common/widgets/dialog.dart:280`。

## 规则

- LOCK：二元设置使用 switch / checkbox，不用纯文本模拟。
- LOCK：危险或连接失败相关弹窗必须有取消或关闭路径。
- GUIDE：网络诊断文本应可复制或可从日志获取。
