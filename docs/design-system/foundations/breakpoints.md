# Breakpoints

## 平台分流

| 场景 | 判断来源 | 规则 |
| --- | --- | --- |
| desktop | `isDesktop` / `isWebDesktop` | 保持紧凑信息密度和 hover 支持 |
| mobile | `isAndroid` / `isIOS` / `isMobile` | 优先触控尺寸和纵向布局 |
| web | `isWeb` | 复用 web 专用 bridge 与页面 |

## 证据

- 平台布尔值定义在 `flutter/lib/common.dart:53` 至 `flutter/lib/common.dart:64`。
- 桌面和移动设置页使用不同文件，见 `flutter/lib/desktop/pages/desktop_setting_page.dart:1`、`flutter/lib/mobile/pages/settings_page.dart:1`。

## 规则

- LOCK：UI 变更必须同时判断 desktop/mobile 影响。
- GUIDE：本次 IPv6 P2P feature 不改变 UI 时，breakpoint 验收写 N/A。
