# Interaction Consistency

## 设置状态单一来源

- LOCK：设置项的 UI 状态必须来自 RustDesk 现有 option 读写路径，不维护第二份本地 truth。
- LOCK：写入设置后重新读取真实 option 值再更新 UI。
- 证据：移动设置页在切换 `kOptionEnableIpv6Punch` 后调用 `mainGetLocalBoolOptionSync`，见 `flutter/lib/mobile/pages/settings_page.dart:804`。

## 网络连接反馈

- LOCK：核心网络功能优先通过 Rust 日志给出诊断，不在 UI 中显示未经验证的 P2P 成功状态。
- GUIDE：如果未来增加诊断页，状态应区分 STUN、interface fallback、IPv6 UDP、IPv4 UDP、TCP punch、relay。

## 翻译与文案

- LOCK：可见文案使用 `translate()` 和现有语言表。
- GUIDE：开发诊断文案保持短句，避免解释性长段落占据设置页。
