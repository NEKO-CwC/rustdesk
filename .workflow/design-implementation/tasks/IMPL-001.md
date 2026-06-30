# IMPL-001 — IPv6 候选分类与接口枚举基础

type: foundation
status: complete
_Refs: REQ-1.1, REQ-1.2, REQ-1.3, REQ-1.4, REQ-6.2_

## 目标

在 `src/common.rs` 中实现可单测的接口 IPv6 候选分类、过滤和确定性选择基础逻辑。

## 上下文

- 相关设计章节 / 交接文件：`design.md#IPv6 Candidate Classifier`、`design.md#Interface Candidate Provider`、`design-handoff.md#代码范围`。
- design-repo refs：`docs/design-system/design-repo.md#FREE-1`、`docs/design-system/foundations/motion.md#原语`。
- 可复用的现有模式：`src/lan.rs` 已使用 `default_net::get_interfaces()`；MVP 不新增依赖。
- 仓库证据：`Cargo.toml:69`、`src/lan.rs:87`、`src/lan.rs:125`、`src/lan.rs:168`、`src/common.rs:95`、`src/common.rs:2418`、`src/common.rs:2448`。

## 步骤

1. 在 `src/common.rs` 提取纯函数，判断 `Ipv6Addr` 是否允许作为公网 P2P 候选。
2. 过滤 loopback、unspecified、multicast、unique-local、link-local 和不在 `2000::/3` 的地址。
3. 新增一个内部候选收集函数，在非 iOS 平台使用 `default_net::get_interfaces()` 收集 IPv6 地址；iOS 路径返回空候选并保持 STUN-only 行为。
4. 为多个候选定义确定性排序或选择规则，并在代码注释中说明该规则的稳定性边界。
5. 同步添加单元测试覆盖公网、非公网、稳定地址、临时隐私地址和多候选排序。

## 范围

- include_paths:
  - `src/common.rs`
- exclude_paths:
  - `libs/hbb_common/protos/rendezvous.proto`
  - `src/client.rs`
  - `src/rendezvous_mediator.rs`
  - `flutter/`
- allowed_changes:
  - 新增私有 helper 函数。
  - 新增 `#[cfg(test)]` 单元测试。
  - 使用已有 `default_net` 依赖。
- non_goals:
  - 不修改协议字段。
  - 不绑定 UDP socket。
  - 不改变 `PUBLIC_IPV6_ADDR` 写入路径。
  - 不新增 UI 或配置项。

## 验收

1. WHEN 输入为 loopback、unspecified、multicast、unique-local、link-local 或非 `2000::/3` IPv6 地址 THEN the system SHALL 拒绝该地址作为接口来源公网候选 _(REQ-1.2; verification_modality: static)_。
2. WHEN 输入为 `2000::/3` 范围内的全局 IPv6 地址 THEN the system SHALL 允许其进入候选列表 _(REQ-1.1, REQ-6.2; verification_modality: static)_。
3. WHEN 同时存在多个有效全局 IPv6 地址 THEN the system SHALL 使用确定性规则输出稳定顺序或稳定首选候选 _(REQ-1.3; verification_modality: static)_。
4. IF 目标平台是 iOS THEN the system SHALL 不调用 `default_net::get_interfaces()` 的接口枚举路径 _(REQ-1.4; verification_modality: static)_。

## TDD

- 先写或同步编写的测试：
  - `test_ipv6_candidate_rejects_non_global_addresses`
  - `test_ipv6_candidate_accepts_global_addresses`
  - `test_ipv6_candidate_selection_is_deterministic`
- 测试命令：
  - `cargo test --locked --lib ipv6_candidate`

## 验证

- 命令：
  - `VCPKG_ROOT=/Users/neko/vcpkg PATH=/Users/neko/.local/homebrew-tools/nasm/3.02/bin:/Users/neko/.local/homebrew-tools/yasm/1.3.0_2/bin:$PATH cargo test --locked --lib ipv6_candidate`：通过，3 个候选分类测试通过。
  - `VCPKG_ROOT=/Users/neko/vcpkg PATH=/Users/neko/.local/homebrew-tools/nasm/3.02/bin:/Users/neko/.local/homebrew-tools/yasm/1.3.0_2/bin:$PATH cargo test --locked --lib`：通过，73 个 lib 测试通过。
- requires_runtime: false
- runtime_entry: N/A
- runtime_rebuild_command: N/A
- verification_modality: static
- component_reuse: N/A
- ux_quality_dimensions: N/A
- screenshot_targets: N/A
- burst_targets: N/A

## 接线检查

- 接入对象：后续 `IMPL-002` 的 fallback bind 逻辑。
- 无孤立代码：yes，helper 必须由 `IMPL-002` 接入 `test_ipv6()`；本任务完成后允许 helper 暂未改变运行时行为，但必须有测试覆盖。
