# IMPL-003 — 连接路径诊断日志与 socket_addr_v6 可观测性

type: integration
status: complete
_Refs: REQ-3.1, REQ-3.2, REQ-3.3, REQ-5.1, REQ-5.2, REQ-5.3, REQ-5.4, REQ-7.3, REQ-7.4_

## 目标

补足最小诊断日志，使 macOS 和 Windows 双端日志能判断 `socket_addr_v6` 是否准备、发送、回传以及最终连接类型。

## 上下文

- 相关设计章节 / 交接文件：`design.md#Logging`、`design.md#Existing Rendezvous Path`、`design.md#Runtime Verification`、`design-handoff.md#代码范围`。
- design-repo refs：`docs/design-system/design-repo.md#FREE-2`。
- 可复用的现有模式：已有连接类型日志 `used to establish {typ} connection`。
- 仓库证据：`src/client.rs:451`、`src/client.rs:462`、`src/client.rs:471`、`src/client.rs:577`、`src/client.rs:707`、`src/rendezvous_mediator.rs:929`、`src/rendezvous_mediator.rs:936`、`src/rendezvous_mediator.rs:943`。

## 步骤

1. 在 `src/common.rs` 的 IPv6 discovery 路径记录：STUN bind / discovery 失败、interface fallback 开始、候选拒绝原因、候选 bind 成功、无候选可用。
2. 检查 `src/client.rs` 现有日志是否足够判断发起端 `socket_addr_v6` 非空；不足时仅补充布尔型或长度级别日志，不输出完整敏感地址清单。
3. 检查 `src/rendezvous_mediator.rs::start_ipv6()` 现有日志是否足够判断受控端回传非空 `socket_addr_v6`；不足时补充最小日志。
4. 保持最终连接类型日志 `used to establish IPv6 connection` / `Relay connection` 可被 E2E 搜索到。
5. 不改变连接选择逻辑，不把日志任务扩大为行为改动。

## 范围

- include_paths:
  - `src/common.rs`
  - `src/client.rs`
  - `src/rendezvous_mediator.rs`
- exclude_paths:
  - `libs/hbb_common/protos/rendezvous.proto`
  - `flutter/`
  - rustdesk-server / hbbs / hbbr
- allowed_changes:
  - 添加最小 `log::debug!` / `log::info!` / `log::warn!`。
  - 不改变协议，不改变连接优先级。
- non_goals:
  - 不输出完整接口列表。
  - 不输出 server key、token 或敏感配置。
  - 不新增 UI 诊断页。

## 验收

1. WHEN STUN IPv6 discovery 失败且 interface fallback 开始 THEN the system SHALL 输出可搜索的 fallback 路径日志 _(REQ-5.1; verification_modality: static)_。
2. WHEN 接口 IPv6 地址被拒绝 THEN the system SHALL 记录简短拒绝原因，且不输出无关接口元数据 _(REQ-5.2; verification_modality: static)_。
3. WHEN 可绑定接口候选被选中 THEN the system SHALL 记录候选来源为 interface discovery _(REQ-5.3; verification_modality: static)_。
4. IF 没有 IPv6 候选可用 THEN the system SHALL 在继续 fallback 前记录 IPv6 candidate unavailable _(REQ-5.4; verification_modality: static)_。
5. WHEN 发起端发送 punch request 或受控端回传 response THEN the system SHALL 能通过日志判断 `socket_addr_v6` 是否非空，而不改变现有 `socket_addr_v6` 字段语义 _(REQ-3.1, REQ-3.2; verification_modality: static)_。
6. IF peer 未返回有效 IPv6 或 IPv6 UDP 尝试失败 THEN the system SHALL 保持现有并发 UDP/TCP/relay 尝试，并通过最终连接类型日志识别结果 _(REQ-3.3, REQ-7.3, REQ-7.4; verification_modality: static)_。

## TDD

- 先写或同步编写的测试：
  - 日志本身不强制写单元测试；优先通过代码审查和运行时日志 grep 验证。
  - 若新增 discovery result enum，则为 enum 到日志状态映射添加单元测试。
- 测试命令：
  - `cargo test --locked --lib ipv6_candidate`
  - `cargo test --locked --lib interface_ipv6`

## 验证

- 命令：
  - `VCPKG_ROOT=/Users/neko/vcpkg PATH=/Users/neko/.local/homebrew-tools/nasm/3.02/bin:/Users/neko/.local/homebrew-tools/yasm/1.3.0_2/bin:$PATH cargo test --locked --lib`：通过，73 个 lib 测试通过。
  - `VCPKG_ROOT=/Users/neko/vcpkg PATH=/Users/neko/.local/homebrew-tools/nasm/3.02/bin:/Users/neko/.local/homebrew-tools/yasm/1.3.0_2/bin:$PATH cargo build --locked --lib`：通过。
  - `rg "interface fallback|socket_addr_v6|IPv6 candidate|used to establish" src/common.rs src/client.rs src/rendezvous_mediator.rs`：通过，能定位 interface fallback、候选不可用、发起端 / 受控端 `socket_addr_v6` 非空与最终连接类型日志。
- requires_runtime: false
- runtime_entry: N/A
- runtime_rebuild_command: N/A
- verification_modality: static
- component_reuse: N/A
- ux_quality_dimensions: N/A
- screenshot_targets: N/A
- burst_targets: N/A

## 接线检查

- 接入对象：`IMPL-002` 的 fallback 行为；现有发起端和受控端 rendezvous 路径。
- 无孤立代码：yes，日志必须挂在真实 discovery / send / receive / connect 路径上。
