# Iconography

## 标准来源

- Flutter Material `Icons`：设置页、远程工具栏、菜单项默认图标来源。
- `IconFont`：仓库自定义 tabbar、搜索、地址簿、设备组等图标，见 `flutter/lib/common.dart:119`。

## 规则

- LOCK：功能按钮使用 `Icons` 或 `IconFont`。
- LOCK：不得使用 emoji 作为功能图标。
- GUIDE：不熟悉的图标应配合 tooltip 或可见文本。
