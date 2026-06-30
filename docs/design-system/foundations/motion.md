# Motion

## 原语

| id | 名称 | 用法 | reduced-motion |
| --- | --- | --- | --- |
| `MOTION-M1` | none | 核心网络状态、日志、非 UI 变更 | 保持无动画 |
| `MOTION-M2` | hover-feedback | 桌面按钮 hover / pressed 反馈 | 保留颜色状态，取消过渡时长 |
| `MOTION-M3` | overlay-entry | 弹窗、菜单、toast 的系统默认进入/退出 | 使用 Flutter 默认简化过渡 |

## 证据

- 桌面按钮有 hover 和 pressed 状态，见 `flutter/lib/desktop/widgets/button.dart:41`。
- Tooltip 等待时间由主题统一定义，见 `flutter/lib/common.dart:311`。

## 规则

- LOCK：不涉及 UI 的 networking feature 在 per-feature `design.md` 写 `Motion: N/A`。
- LOCK：声明 motion 的 feature 必须引用本文件的 `MOTION-Mx`，并在验收中提供 temporal 证据。
