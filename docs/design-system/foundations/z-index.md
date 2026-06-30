# Z-Index And Overlay Order

Flutter 使用路由、`Overlay`、`OverlayEntry`、`showMenu` 和 BotToast 管理层叠。本文件定义仓库级逻辑层级，避免页面局部层级冲突。

## 层叠量表

| layer | order | 参与者 | 规则 |
| --- | ---: | --- | --- |
| `base` | 0 | 普通页面内容、远程画布 | 页面默认内容 |
| `sticky-table-header` | 10 | 表头、固定工具栏 | 不得遮挡 dropdown / popover |
| `sticky-table-column` | 20 | 固定列、固定操作区 | 不得遮挡 dropdown / popover |
| `dropdown-popover` | 30 | `PopupMenuItem`、`showMenu`、select panel、field popover、combobox | 必须高于 sticky 元素 |
| `tooltip` | 40 | `Tooltip` | 高于 dropdown，但不阻塞主要操作 |
| `drawer-sheet` | 50 | 抽屉、sheet、移动端动作面板 | 必须有可达 dismiss |
| `modal` | 60 | `CustomAlertDialog`、普通对话框 | 高于 drawer |
| `confirm` | 70 | 危险确认、权限确认 | 高于普通 modal |
| `toast` | 80 | BotToast、短暂状态提示 | 非阻塞，避免遮挡确认按钮 |

## Overlay 契约

- LOCK：modal、confirm、dropdown、select panel、field popover、combobox、menu、drawer、sheet、tooltip、toast 都必须引用本层级。
- LOCK：所有 overlay 必须相对 viewport 限高，内容区域内部滚动。
- LOCK：dismiss 或 close 操作在内容滚动时仍可到达。
- LOCK：dropdown / field popover 必须高于 sticky header、sticky column 和固定操作区。
- LOCK：不得引入页面局部 z-index ladder。

## 证据

- 远程移动页直接创建 `Overlay` 和 `OverlayEntry`，见 `flutter/lib/mobile/pages/remote_page.dart:500`。
- 桌面 tab 右键菜单使用 BotToast attached widget，见 `flutter/lib/desktop/widgets/tabbar_widget.dart:75`。
- 移动远程工具菜单使用 `showMenu`，见 `flutter/lib/mobile/pages/remote_page.dart:792`。
