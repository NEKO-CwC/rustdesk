# Implementation Goal - Interface IPv6 Candidate P2P

Status: code_verified_runtime_pending

## 目标

在 RustDesk fork 中落地接口枚举来源的全局 IPv6 P2P 候选 fallback：当 `enable-ipv6-punch` 已启用且现有 STUN / connected-bind IPv6 发现失败时，客户端枚举本机接口 IPv6，选择可绑定的公网 IPv6 地址，通过现有 `socket_addr_v6` 路径参与 P2P 连接，避免无必要 relay。

## 范围锁定

- 使用现有 `enable-ipv6-punch` 开关。
- 使用现有单个 `socket_addr_v6` 协议字段。
- 不修改 rustdesk-server / hbbs / hbbr 协议。
- 不新增 UI。
- 不实现 full ICE、TURN、多 candidate racing。
- 不承诺绕过防火墙、运营商入站限制或无公网 IPv6 的网络环境。

## 成功结果

- `cargo test --locked --lib` 通过。
- `cargo build --locked --lib` 通过。
- macOS 端在 STUN IPv6 失败时能通过 interface fallback 准备非空 `socket_addr_v6`。
- 推送到 `https://github.com/NEKO-CwC/rustdesk` 的 `feat/interface-ipv6-candidate-p2p` 分支。
- Windows 端能运行同 commit fork artifact 并执行多设备 E2E。
- 可直连 IPv6 拓扑下最终连接类型为 `IPv6`，且延迟中位数低于 forced relay baseline。

## 执行约束

- 代码实现必须遵守 `AGENTS.md`：不使用生产路径 `unwrap()` / `expect()`，不新增不必要依赖，不做无关重构。
- 优先在 `src/common.rs` 内完成 IPv6 候选发现、过滤、绑定和日志。
- 只在必要时触碰 `src/client.rs`、`src/rendezvous_mediator.rs` 或 `libs/hbb_common/protos/rendezvous.proto`；MVP 预期不修改 protobuf。
- 任何改变 wire protocol、设置 UI 或 server 行为的需求都必须先重新打开 PRD。

## 当前落地步骤

1. Phase 2：已完成，`design.md` 与 `design-handoff.md` 已冻结。
2. Phase 3：已完成，`tasks/IMPL-*.md` 与 `traceability-matrix.md` 已生成。
3. Phase 4：代码级实现和 macOS 本机验证已完成；Windows 同 commit E2E 待执行。
4. Phase 5：已写 `implementation-handoff.md`，但收敛不能完成，直到 Windows E2E 记录真实连接类型和延迟 baseline。

## 当前验证结果

- `cargo test --locked --lib ipv6_candidate`：通过，3 个测试。
- `cargo test --locked --lib interface_ipv6`：通过，4 个测试。
- `cargo test --locked --lib ipv6`：通过，11 个测试。
- `cargo test --locked --lib`：通过，73 个测试。
- `cargo build --locked --lib`：通过。
- `rendezvous.proto` / Flutter UI：无 diff。

## 推送与 Windows 协作

推送点不是最终验收点，而是跨设备同步点。macOS 本机测试通过后，将 feature branch 推送到 fork，使 Windows 可以获取同 commit artifact 或源码继续验证。

Windows 端不得使用上游 stock RustDesk 作为 feature 验收端；stock RustDesk 只允许作为 relay/direct baseline。
