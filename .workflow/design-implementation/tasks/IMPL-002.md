# IMPL-002 — 接入 test_ipv6 的接口 fallback 与 UDP bind 验证

type: integration
status: complete
_Refs: REQ-1.1, REQ-2.1, REQ-2.2, REQ-2.3, REQ-3.4, REQ-4.1, REQ-4.2, REQ-4.3, REQ-6.1, REQ-6.2_

## 目标

在 `src/common.rs::test_ipv6()` 中接入接口来源 IPv6 fallback，并在发布前完成 UDP bind 验证。

## 上下文

- 相关设计章节 / 交接文件：`design.md#UDP Bind Verifier`、`design.md#State And Control`、`design.md#Existing Rendezvous Path`、`design-handoff.md#实现提示`。
- design-repo refs：`docs/design-system/design-repo.md#FREE-2`。
- 可复用的现有模式：`PUBLIC_IPV6_ADDR` 缓存、`test_bind_ipv6()`、`get_ipv6_socket()`。
- 仓库证据：`src/common.rs:95`、`src/common.rs:1100`、`src/common.rs:1107`、`src/common.rs:2418`、`src/common.rs:2430`、`src/common.rs:2478`、`src/common.rs:2564`。

## 步骤

1. 基于 `IMPL-001` 的候选列表新增内部 async helper：按候选顺序尝试 `UdpSocket::bind(SocketAddr::from((candidate, 0)))`。
2. bind 成功时读取 `socket.local_addr()`，将端口归零后写入 `PUBLIC_IPV6_ADDR`，保持 `get_ipv6_socket()` 继续绑定真实端口并编码 `socket_addr_v6` 的现有语义。
3. bind 失败时记录简短原因并尝试下一个候选。
4. 在 `test_bind_ipv6()` 失败后、STUN async job 仍可继续前，启动 interface fallback；STUN 成功仍保持优先或覆盖现有缓存。
5. 保持 `enable-ipv6-punch` 的现有调用边界：只有调用方在开关开启时才进入 `test_ipv6()` / `get_ipv6_socket()`，不得放宽自建 server 空配置默认禁用行为。
6. 新增或调整测试，使用可注入候选列表验证 bind 成功、bind 失败、无候选 fallback 和最多发布一个 socket address。

## 范围

- include_paths:
  - `src/common.rs`
- exclude_paths:
  - `libs/hbb_common/protos/rendezvous.proto`
  - `flutter/`
  - rustdesk-server / hbbs / hbbr
- allowed_changes:
  - 新增 `src/common.rs` 内部 async helper。
  - 新增测试 helper，只在 `#[cfg(test)]` 下使用。
  - 调整 `test_ipv6()` 内部 fallback 顺序和日志。
- non_goals:
  - 不新增协议字段。
  - 不新增多候选 racing。
  - 不修改现有 `get_ipv6_punch_enabled()` 默认策略。
  - 不让 fallback 失败影响 IPv4 / TCP / relay。

## 验收

1. WHEN `test_bind_ipv6()` 失败且存在可绑定接口来源全局 IPv6 THEN the system SHALL 绑定该候选 UDP port `0` 并准备一个可由 `get_ipv6_socket()` 发布的 IPv6 socket address _(REQ-1.1, REQ-2.1, REQ-6.2; verification_modality: static)_。
2. IF 某个接口候选 bind 失败 THEN the system SHALL 拒绝该候选、记录失败原因并尝试下一个有效候选 _(REQ-2.2; verification_modality: static)_。
3. IF 没有任何接口来源候选可绑定 THEN the system SHALL 保持 `socket_addr_v6` 为空并继续现有 fallback 流程 _(REQ-2.3; verification_modality: static)_。
4. WHEN STUN IPv6 discovery 成功 THEN the system SHALL 保持 STUN-derived 行为优先，不被 interface fallback 强制覆盖 _(REQ-3.4; verification_modality: static)_。
5. WHEN `enable-ipv6-punch` 未启用或自建 server 空配置默认禁用 THEN the system SHALL 不因本任务改变现有开关边界 _(REQ-4.1, REQ-4.2, REQ-4.3, REQ-6.1; verification_modality: static)_。

## TDD

- 先写或同步编写的测试：
  - `test_interface_ipv6_bind_fallback_uses_first_bindable_candidate`
  - `test_interface_ipv6_bind_fallback_skips_failed_bind`
  - `test_interface_ipv6_bind_fallback_returns_none_without_candidates`
  - `test_interface_ipv6_fallback_keeps_single_candidate`
- 测试命令：
  - `cargo test --locked --lib interface_ipv6`

## 验证

- 命令：
  - `VCPKG_ROOT=/Users/neko/vcpkg PATH=/Users/neko/.local/homebrew-tools/nasm/3.02/bin:/Users/neko/.local/homebrew-tools/yasm/1.3.0_2/bin:$PATH cargo test --locked --lib interface_ipv6`：通过，4 个 interface fallback bind 测试通过。
  - `VCPKG_ROOT=/Users/neko/vcpkg PATH=/Users/neko/.local/homebrew-tools/nasm/3.02/bin:/Users/neko/.local/homebrew-tools/yasm/1.3.0_2/bin:$PATH cargo test --locked --lib ipv6`：通过，11 个 IPv6 相关测试通过，包含 connected candidate 被拒后进入 fallback 的分支。
  - `VCPKG_ROOT=/Users/neko/vcpkg PATH=/Users/neko/.local/homebrew-tools/nasm/3.02/bin:/Users/neko/.local/homebrew-tools/yasm/1.3.0_2/bin:$PATH cargo test --locked --lib`：通过，73 个 lib 测试通过。
  - `VCPKG_ROOT=/Users/neko/vcpkg PATH=/Users/neko/.local/homebrew-tools/nasm/3.02/bin:/Users/neko/.local/homebrew-tools/yasm/1.3.0_2/bin:$PATH cargo build --locked --lib`：通过。
- requires_runtime: false
- runtime_entry: N/A
- runtime_rebuild_command: N/A
- verification_modality: static
- component_reuse: N/A
- ux_quality_dimensions: N/A
- screenshot_targets: N/A
- burst_targets: N/A

## 接线检查

- 接入对象：`IMPL-001` 的候选分类与选择 helper；`src/common.rs::test_ipv6()`；`src/common.rs::get_ipv6_socket()`。
- 无孤立代码：yes，fallback helper 必须被 `test_ipv6()` 调用，测试必须覆盖成功和失败路径。
