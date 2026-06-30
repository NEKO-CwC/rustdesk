# IMPL-004 — macOS 构建验证与 Windows 同 commit E2E 交接

type: verification
status: in_progress
_Refs: REQ-7.1, REQ-7.2, REQ-7.3, REQ-7.4, REQ-7.5, REQ-7.6_

## 目标

完成 macOS 侧代码验证，并准备推送 fork 与 Windows 同 commit 多设备 E2E 的可执行交接。

## 上下文

- 相关设计章节 / 交接文件：`design.md#Runtime Verification`、`design-handoff.md#验证命令`、`development-pipeline.md`、`docs/project-map/verification.md#cap-1`。
- design-repo refs：`docs/design-system/conformance/acceptance.md#Gate 5 - 视觉回归`，截图 N/A。
- 可复用的现有模式：CI 使用 `cargo test --locked`，Windows portable build 由 `build.py` 支持。
- 仓库证据：`.github/workflows/ci.yml:246`、`.github/workflows/ci.yml:250`、`build.py:405`、`build.py:409`、`build.py:440`、`build.py:447`、`.github/workflows/flutter-build.yml:256`、`.github/workflows/flutter-build.yml:258`。

## 步骤

1. 在 macOS 本机运行 `cargo test --locked --lib` 和 `cargo build --locked --lib`，记录结果。
2. 确认本地 branch 为 `feat/interface-ipv6-candidate-p2p`，remote `neko` 指向 `https://github.com/NEKO-CwC/rustdesk.git`。
3. 在 macOS 能构建或运行客户端时，收集日志证据：STUN IPv6 失败、interface fallback、非空 `socket_addr_v6` 或失败原因。
4. 准备推送 fork 的 commit hash 和命令。
5. 将 Windows 端继续测试 prompt、artifact / commit 要求、E2E 表格字段写入 `implementation-handoff.md` 或 Phase 4 验收记录。
6. 运行或交接多设备 E2E：5 次 direct path，3 次 forced relay baseline。

## 范围

- include_paths:
  - `.workflow/design-implementation/implementation-log.md`
  - `.workflow/design-implementation/implementation-handoff.md`
  - 源码实现涉及的 Rust 文件
- exclude_paths:
  - UI 截图目录，除非后续新增 UI。
  - rustdesk-server / hbbs / hbbr
- allowed_changes:
  - 更新验证记录。
  - 添加 fork remote 或创建 feature branch。
  - 推送 feature branch 到 `neko` remote。
- non_goals:
  - 不把 Windows 人工 E2E 伪装成 macOS 本机验证。
  - 不用 stock RustDesk 作为 feature 验收端。
  - 不在缺少同 commit Windows artifact 时标记多设备 E2E 通过。

## 验收

1. WHEN 执行本 feature 的多设备 E2E THEN the system SHALL 要求 macOS 和 Windows 两端运行同一个 fork commit，并记录 commit hash、artifact 路径和启动时间 _(REQ-7.1; verification_modality: runtime)_。
2. IF Windows 设备没有本地构建环境 THEN the system SHALL 允许 Windows 运行 CI 或另一台机器生成的同 commit artifact，且 stock RustDesk 只作为 baseline _(REQ-7.2; verification_modality: runtime)_。
3. WHEN E2E 处于可 IPv6 直连拓扑 THEN the system SHALL 以最终连接类型 `IPv6` 作为 direct-path 通过条件 _(REQ-7.3; verification_modality: runtime)_。
4. WHEN E2E 建立 IPv6 direct 连接 THEN the system SHALL 记录连接建立耗时、会话延迟样本、候选来源、是否出现 relay request，并与 forced relay baseline 对比 _(REQ-7.4; verification_modality: runtime)_。
5. IF IPv6 direct 延迟中位数不低于 forced relay baseline THEN the system SHALL 将性能验收标为 failed 或 blocked，并记录可能原因 _(REQ-7.5; verification_modality: runtime)_。
6. WHEN macOS 侧 STUN IPv6 DNS / route 被 Clash Party 或 mihomo 阻断 THEN the system SHALL 仍通过 interface fallback 准备非空 `socket_addr_v6`，并在 Windows 回传本端 `socket_addr_v6` 后继续 IPv6 direct 尝试 _(REQ-7.6; verification_modality: runtime)_。

## TDD

- 先写或同步编写的测试：
  - 本任务不新增自动化单元测试；它执行前置任务的测试和真实运行时验证。
- 测试命令：
  - `cargo test --locked --lib`
  - `cargo build --locked --lib`

## 验证

- 命令：
  - `VCPKG_ROOT=/Users/neko/vcpkg PATH=/Users/neko/.local/homebrew-tools/nasm/3.02/bin:/Users/neko/.local/homebrew-tools/yasm/1.3.0_2/bin:$PATH cargo test --locked --lib`：通过，73 个 lib 测试通过。
  - `VCPKG_ROOT=/Users/neko/vcpkg PATH=/Users/neko/.local/homebrew-tools/nasm/3.02/bin:/Users/neko/.local/homebrew-tools/yasm/1.3.0_2/bin:$PATH cargo build --locked --lib`：通过。
  - `git remote get-url neko`：通过，`https://github.com/NEKO-CwC/rustdesk.git`。
  - `git rev-parse HEAD`：推送后以 fork 分支 tip commit 为 Windows E2E 同步点。
- requires_runtime: true
- runtime_entry:
  - `AGENTS.md#RustDesk Guide`
  - `docs/project-map/verification.md#cap-1`
  - `.workflow/design-implementation/development-pipeline.md`
- runtime_rebuild_command:
  - macOS code path: `cargo build --locked --lib`
  - full desktop artifact: follow `build.py` platform command if local dependencies are available
- verification_modality: runtime
- component_reuse: N/A
- ux_quality_dimensions: N/A
- screenshot_targets: N/A
- burst_targets: N/A

## 接线检查

- 接入对象：`IMPL-001`、`IMPL-002`、`IMPL-003` 的代码和日志结果。
- 无孤立代码：partial，macOS 代码级验证已通过；Windows 同 commit direct-path E2E、forced relay baseline、延迟对比仍待执行，不得标记为通过。
