---
status: active
version: 0.1.0
last_reviewed: 2026-06-30
---

# RustDesk 二开项目地图

## 项目目的与范围

本项目是在 RustDesk 上进行二次开发，当前地图只覆盖本次网络连接增强所需的最小能力边界：远程连接建立、P2P 候选准备、rendezvous 协议字段、以及失败后的 relay 回退。它不是 RustDesk 全量产品地图。

## 角色清单

- `user`：使用 RustDesk 发起或接受远程连接的人。
- `operator`：配置自建 ID server、relay server、客户端连接策略的人。
- `maintainer`：维护 RustDesk 二开代码、验证连接路径和发布构建的人。

## 能力索引

- `CAP-1`：远程 P2P 连接建立，见 [capabilities.md](capabilities.md)。

## 验证索引

- `CAP-1` 的验证规格见 [verification.md#cap-1](verification.md#cap-1)。

## 维护纪律

能力、角色、模块依赖和验证引用只在 `docs/project-map/` 中维护。单个 feature 的 PRD 只引用能力 id 和 `verification_ref`，不复制完整验证规格。
