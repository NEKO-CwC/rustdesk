# Terminology

## 标准术语

| 概念 | 中文显示 | 代码标识 |
| --- | --- | --- |
| IPv6 P2P | 启用 IPv6 P2P 连接 | `enable-ipv6-punch` |
| UDP hole punching | 启用 UDP 打洞 | `enable-udp-punch` |
| ID server | ID server | rendezvous server |
| relay | relay | relay |
| socket_addr_v6 | IPv6 socket 地址 | `socket_addr_v6` |

## 证据

- 简体中文语言表已有“启用 IPv6 P2P 连接”，见 `src/lang/cn.rs:693`。
- Flutter option 常量为 `kOptionEnableIpv6Punch`，见 `flutter/lib/consts.dart:171`。

## 规则

- LOCK：用户可见设置沿用现有翻译，不把 `socket_addr_v6` 暴露成普通用户文案。
- GUIDE：面向 maintainer 的日志可以使用 `socket_addr_v6`、STUN、interface fallback 等技术词。
