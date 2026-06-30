# Traceability Matrix - Interface IPv6 Candidate P2P

Status: phase4_code_verified_runtime_pending

## 验证摘要

- 代码级验证已通过：`ipv6_candidate`、`interface_ipv6`、全量 `cargo test --locked --lib`、`cargo build --locked --lib`。
- 日志与协议范围检查已通过：保留现有 `socket_addr_v6` 单字段，不修改 protobuf / server / Flutter UI。
- 多设备 E2E 未完成：Windows 端必须运行同 commit fork artifact 后，再验证最终连接类型、UDP/IPv6 direct 和延迟 baseline。

| REQ | 设计章节 | 任务 | 测试 / 验证 | 状态 |
| --- | --- | --- | --- | --- |
| REQ-1.1 | `design.md#IPv6 Candidate Classifier`, `design.md#Interface Candidate Provider` | IMPL-001, IMPL-002 | `cargo test --locked --lib ipv6_candidate`, `cargo test --locked --lib interface_ipv6` | passed-code |
| REQ-1.2 | `design.md#IPv6 Candidate Classifier` | IMPL-001 | `test_ipv6_candidate_rejects_non_global_addresses` | passed-code |
| REQ-1.3 | `design.md#IPv6 Candidate Classifier`, `design.md#Interface Candidate Provider` | IMPL-001 | `test_ipv6_candidate_selection_is_deterministic` | passed-code |
| REQ-1.4 | `design.md#Interface Candidate Provider` | IMPL-001 | iOS cfg gate code review + `cargo test --locked --lib` | passed-code |
| REQ-2.1 | `design.md#UDP Bind Verifier` | IMPL-002 | `test_interface_ipv6_bind_fallback_uses_first_bindable_candidate` | passed-code |
| REQ-2.2 | `design.md#UDP Bind Verifier` | IMPL-002 | `test_interface_ipv6_bind_fallback_skips_failed_bind` | passed-code |
| REQ-2.3 | `design.md#UDP Bind Verifier` | IMPL-002 | `test_interface_ipv6_bind_fallback_returns_none_without_candidates` | passed-code |
| REQ-3.1 | `design.md#Existing Rendezvous Path`, `design.md#Logging` | IMPL-003 | `cargo build --locked --lib`,发起端 `socket_addr_v6` 日志检查 | passed-code |
| REQ-3.2 | `design.md#Existing Rendezvous Path`, `design.md#Logging` | IMPL-003 | `cargo build --locked --lib`,受控端 `socket_addr_v6` 日志检查 | passed-code |
| REQ-3.3 | `design.md#Existing Rendezvous Path`, `design.md#Logging` | IMPL-003, IMPL-004 | final connection type log path present；真实连接结果待 Windows E2E | partial-runtime |
| REQ-3.4 | `design.md#Existing Rendezvous Path` | IMPL-002 | STUN async discovery 仍可覆盖 fallback 缓存 + `cargo test --locked --lib` | passed-code |
| REQ-4.1 | `design.md#State And Control` | IMPL-002 | switch boundary code review：调用方仍由 `get_ipv6_punch_enabled()` 控制 | passed-code |
| REQ-4.2 | `design.md#State And Control` | IMPL-002 | `get_local_option()` 默认行为未修改 | passed-code |
| REQ-4.3 | `design.md#State And Control` | IMPL-002 | 显式启用 IPv6 punch 后允许进入 fallback 的现有路径未收窄 | passed-code |
| REQ-4.4 | `design.md#Forbidden Patterns` | IMPL-002 | no Flutter/UI diff | passed-code |
| REQ-5.1 | `design.md#Logging` | IMPL-003 | `rg "Starting interface IPv6 fallback" src/common.rs` | passed-code |
| REQ-5.2 | `design.md#Logging` | IMPL-003 | rejected candidate reason log review | passed-code |
| REQ-5.3 | `design.md#Logging` | IMPL-003 | selected candidate source log review | passed-code |
| REQ-5.4 | `design.md#Logging` | IMPL-003 | unavailable candidate log review | passed-code |
| REQ-6.1 | `design.md#State And Control`, `design.md#Forbidden Patterns` | IMPL-002 | switch boundary code review：未在禁用路径新增发送点 | passed-code |
| REQ-6.2 | `design.md#IPv6 Candidate Classifier`, `design.md#UDP Bind Verifier` | IMPL-001, IMPL-002 | single candidate tests + existing single `socket_addr_v6` path | passed-code |
| REQ-6.3 | `design.md#Forbidden Patterns` | IMPL-003 | no protobuf diff | passed-code |
| REQ-7.1 | `design.md#Runtime Verification` | IMPL-004 | commit hash + artifact record | pending-runtime |
| REQ-7.2 | `design.md#Runtime Verification` | IMPL-004 | Windows artifact/source path record | pending-runtime |
| REQ-7.3 | `design.md#Runtime Verification`, `design.md#Logging` | IMPL-003, IMPL-004 | final connection type `IPv6` logs | pending-runtime |
| REQ-7.4 | `design.md#Runtime Verification`, `design.md#Logging` | IMPL-003, IMPL-004 | latency sample and relay request record | pending-runtime |
| REQ-7.5 | `design.md#Runtime Verification` | IMPL-004 | direct vs forced relay baseline comparison | pending-runtime |
| REQ-7.6 | `design.md#Runtime Verification` | IMPL-002, IMPL-003, IMPL-004 | STUN-blocked interface fallback runtime logs | pending-runtime |

## 已执行命令

```bash
VCPKG_ROOT=/Users/neko/vcpkg PATH=/Users/neko/.local/homebrew-tools/nasm/3.02/bin:/Users/neko/.local/homebrew-tools/yasm/1.3.0_2/bin:$PATH cargo test --locked --lib ipv6_candidate
VCPKG_ROOT=/Users/neko/vcpkg PATH=/Users/neko/.local/homebrew-tools/nasm/3.02/bin:/Users/neko/.local/homebrew-tools/yasm/1.3.0_2/bin:$PATH cargo test --locked --lib interface_ipv6
VCPKG_ROOT=/Users/neko/vcpkg PATH=/Users/neko/.local/homebrew-tools/nasm/3.02/bin:/Users/neko/.local/homebrew-tools/yasm/1.3.0_2/bin:$PATH cargo test --locked --lib ipv6
VCPKG_ROOT=/Users/neko/vcpkg PATH=/Users/neko/.local/homebrew-tools/nasm/3.02/bin:/Users/neko/.local/homebrew-tools/yasm/1.3.0_2/bin:$PATH cargo test --locked --lib
VCPKG_ROOT=/Users/neko/vcpkg PATH=/Users/neko/.local/homebrew-tools/nasm/3.02/bin:/Users/neko/.local/homebrew-tools/yasm/1.3.0_2/bin:$PATH cargo build --locked --lib
rg "interface fallback|socket_addr_v6|IPv6 candidate|used to establish" src/common.rs src/client.rs src/rendezvous_mediator.rs
git diff -- libs/hbb_common/protos/rendezvous.proto flutter
```

## 未关闭项

- Windows 同 commit artifact 运行。
- macOS -> Windows direct path 连续 5 次。
- forced relay baseline 3 次。
- IPv6 direct 延迟中位数与 forced relay baseline 对比。
