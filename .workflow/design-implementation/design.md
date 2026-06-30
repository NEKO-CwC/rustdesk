# Design - Interface IPv6 Candidate P2P

Status: frozen

## Direction

本设计实现一个无 UI 的核心网络能力：当现有 STUN / connected-bind IPv6 发现失败时，在 `enable-ipv6-punch` 已启用的前提下，从本机接口枚举可绑定的全局 IPv6 地址，并复用 RustDesk 现有单个 `socket_addr_v6` 路径参与 P2P 连接。

_Refs: REQ-1, REQ-2, REQ-3, REQ-4, REQ-6_

design_repo_refs:

- `docs/design-system/design-repo.md#FREE-1`
- `docs/design-system/design-repo.md#FREE-2`
- `docs/design-system/foundations/motion.md`
- `docs/design-system/conformance/acceptance.md`

## Component Model

### IPv6 Candidate Classifier

新增或提取纯函数，判断 IPv6 地址是否允许作为公网 P2P 候选。

规则：

- 拒绝 loopback、unspecified、multicast、unique-local、link-local。
- 拒绝不在 `2000::/3` 的地址。
- 允许公网稳定地址和公网临时隐私地址。
- 逻辑必须可单元测试，不依赖真实网卡。

_Refs: REQ-1.1, REQ-1.2, REQ-1.3, REQ-6.2_

### Interface Candidate Provider

在非 iOS 构建目标上复用现有 `default_net::get_interfaces()` 枚举本机接口地址。

规则：

- 不新增依赖。
- 只输出 IPv6 地址候选，不输出无关接口元数据。
- iOS 或存在构建风险的平台保持当前 STUN-only 行为。
- 多个有效候选使用确定性选择规则。

_Refs: REQ-1.1, REQ-1.3, REQ-1.4_

### UDP Bind Verifier

接口来源候选必须在发送给对端前完成 UDP bind 验证。

规则：

- 对候选 IPv6 地址绑定端口 `0`。
- 使用 `socket.local_addr()` 取得真实本地端口。
- bind 失败时记录简短原因并尝试下一个候选。
- 所有候选失败时，保持 `socket_addr_v6` 为空，并继续现有 fallback。

_Refs: REQ-2.1, REQ-2.2, REQ-2.3_

### Existing Rendezvous Path

MVP 不修改 protobuf，不修改 hbbs / hbbr，不引入 candidate list。

规则：

- 发起端继续通过 `PunchHoleRequest.socket_addr_v6` 发送单个候选。
- 受控端继续通过现有 `start_ipv6()` 和 response `socket_addr_v6` 路径处理。
- STUN IPv6 成功时保持当前 STUN-derived 行为优先。
- IPv6 失败时继续现有 IPv4 UDP、TCP punch、relay 并发和 fallback 逻辑。

_Refs: REQ-3.1, REQ-3.2, REQ-3.3, REQ-3.4_

## State And Control

本功能只受现有 `enable-ipv6-punch` 控制。

规则：

- `enable-ipv6-punch` 禁用时，不枚举接口 IPv6，不发布接口来源 `socket_addr_v6`。
- 自建 rendezvous server 且 `enable-ipv6-punch` 未显式启用时，保留当前默认禁用行为。
- 不新增 UI 开关。

_Refs: REQ-4.1, REQ-4.2, REQ-4.3, REQ-4.4, REQ-6.1_

## Logging

日志用于跨设备 E2E 诊断，不展示到 UI。

必须能区分：

- STUN IPv6 成功。
- STUN IPv6 失败并进入 interface fallback。
- 接口地址被拒绝及简短原因。
- 接口候选 bind 成功并被选中。
- 没有可用 IPv6 候选。
- 最终连接类型和连接建立耗时沿用或补足现有日志。

日志不得输出无关接口元数据，不得扩大地址清单暴露面。

_Refs: REQ-5.1, REQ-5.2, REQ-5.3, REQ-5.4, REQ-7.4_

## Motion

Motion: N/A。本 feature 不新增 UI、动画或屏幕状态切换；核心网络行为使用日志和运行时验证。

design_repo_refs:

- `docs/design-system/foundations/motion.md#原语`
- `MOTION-M1`

## Screenshot

Screenshot: N/A。本 feature 不新增页面、不修改现有设置页，也不改变可视组件状态。

design_repo_refs:

- `docs/design-system/conformance/acceptance.md#Gate 5 - 视觉回归`

## Runtime Verification

运行时验证必须使用真实 macOS + Windows 双端 RustDesk 客户端。

要求：

- 双端运行同一个 fork commit。
- macOS 可触发 STUN IPv6 DNS / route 失败但仍保留基础 IPv6 出站。
- Windows 允许 RustDesk UDP 入站和出站。
- 双端配置同一自建 ID server / relay server / key。
- 最终连接类型为 `IPv6` 才能通过 direct-path E2E。
- IPv6 direct 延迟中位数必须低于同拓扑 forced relay baseline。

_Refs: REQ-7.1, REQ-7.2, REQ-7.3, REQ-7.4, REQ-7.5, REQ-7.6_

## Forbidden Patterns

- 不新增可见 UI 开关或诊断页。
- 不修改 `rendezvous.proto` 添加多候选字段。
- 不修改 rustdesk-server 协议。
- 不在 IPv6 P2P 禁用时发送接口来源 IPv6 地址。
- 不用 interface fallback 覆盖 STUN 成功结果。
- 不因为接口枚举失败阻断现有 relay fallback。

_Refs: REQ-3.4, REQ-4.4, REQ-6.1, REQ-6.2, REQ-6.3_
