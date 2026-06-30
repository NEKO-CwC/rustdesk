# Design Handoff - Interface IPv6 Candidate P2P

Status: frozen

## 交接摘要

本 feature 是无 UI 的 RustDesk 核心网络改动。实现应集中在 `src/common.rs` 的 IPv6 探测和 UDP socket 准备路径：当 STUN / connected-bind IPv6 失败时，枚举本机接口 IPv6，筛选全局地址，绑定 UDP socket，复用现有 `socket_addr_v6` 单候选路径。

## 代码范围

首选修改：

- `src/common.rs`

可能涉及：

- `src/client.rs`：仅当现有连接日志不足以识别最终 IPv6 / Relay 类型时补充最小日志。
- `src/rendezvous_mediator.rs`：仅当受控端日志不足以验证 `socket_addr_v6` 回传时补充最小日志。

不应修改：

- `libs/hbb_common/protos/rendezvous.proto`
- rustdesk-server / hbbs / hbbr 协议
- Flutter 设置页

## 实现提示

- 优先提取纯函数：IPv6 地址分类、候选排序或选择。
- 接口枚举使用已有 `default_net::get_interfaces()`。
- 对平台做保守 gating，iOS 维持当前行为。
- bind 验证必须用 UDP socket 绑定候选 IPv6 + port `0`，再读取 `local_addr()`。
- 不能在生产代码中使用不必要的 `unwrap()` / `expect()`。
- bind 或枚举失败必须继续现有 fallback，不得让连接流程提前失败。

## 验证命令

静态 / 集成：

```bash
cargo test --locked --lib
cargo build --locked --lib
```

运行时：

- macOS RustDesk fork build 连接 Windows RustDesk fork build。
- 双端必须是同一个 commit。
- 运行步骤见 `development-pipeline.md` 与 `docs/project-map/verification.md#cap-1`。

## 验收锚点

- REQ-1：接口 IPv6 候选发现与过滤。
- REQ-2：候选 bind 验证与 `socket_addr_v6` 发布。
- REQ-3：复用现有 P2P 语义。
- REQ-4：尊重现有开关。
- REQ-5：可诊断日志。
- REQ-6：隐私与兼容性边界。
- REQ-7：多设备 E2E。

## UI / Motion / Screenshot

- UI: N/A
- Motion: N/A
- Screenshot: N/A

对应 design refs：

- `docs/design-system/design-repo.md#FREE-1`
- `docs/design-system/design-repo.md#FREE-2`
- `docs/design-system/foundations/motion.md`
- `docs/design-system/conformance/acceptance.md`
