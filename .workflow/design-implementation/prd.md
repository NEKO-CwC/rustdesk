# PRD - Interface IPv6 Candidate P2P

Status: frozen

## 简介

本功能为 RustDesk 的现有 IPv6 P2P 路径增加“本机接口枚举得到全局 IPv6 候选”的 fallback。当前 RustDesk 已有 UDP punch、IPv6 P2P 开关和 `socket_addr_v6` 协议字段，但本地 IPv6 候选主要依赖 STUN / DNS 相关路径准备；在 macOS + Clash Party 或 mihomo TUN 场景中，STUN IPv6 解析或路由失败时，客户端可能无法发布 IPv6 候选，进而走 relay/TCP，即使两端实际拥有可用公网 IPv6。MVP 复用现有单个 `socket_addr_v6` 字段，不修改 rustdesk-server 协议。

## Code Map（代码与运行位置）

- 框架 / 路由：桌面客户端是 Rust 2021 crate，package 名为 `rustdesk`，library target 为 `librustdesk`；核心模块从 `src/lib.rs` 引入 `client`、`lan`、`rendezvous_mediator`、`common`。证据：`Cargo.toml:1`、`Cargo.toml:11`、`src/lib.rs:14`、`src/lib.rs:15`、`src/lib.rs:17`、`src/lib.rs:21`。
- 组件模式：IPv6 探测、候选缓存和 UDP socket 准备在 `src/common.rs`，包括 `PUBLIC_IPV6_ADDR`、`test_ipv6()`、`get_ipv6_socket()`。证据：`src/common.rs:95`、`src/common.rs:2418`、`src/common.rs:2564`。
- 数据 / 状态集成：现有本地开关是 `enable-ipv6-punch`；Rust 通过 `get_ipv6_punch_enabled()` 读取，Flutter 常量是 `kOptionEnableIpv6Punch`，自建 rendezvous server 下 UDP/IPv6 punch 的空配置默认返回 `"N"`。证据：`src/common.rs:1100`、`src/common.rs:1107`、`src/common.rs:1109`、`src/common.rs:1111`、`flutter/lib/consts.dart:171`。
- 发起端连接路径：发起端在 IPv6 punch 开启时调用 `test_ipv6()`，再通过 `get_ipv6_socket()` 准备 `socket_addr_v6`，并在 `PunchHoleRequest` 中发送。证据：`src/client.rs:311`、`src/client.rs:451`、`src/client.rs:452`、`src/client.rs:462`、`src/client.rs:471`。
- 受控端连接路径：受控端解码 peer `socket_addr_v6`，调用 `start_ipv6()`，并把本端 `socket_addr_v6` 放回 punch 或 relay response。证据：`src/rendezvous_mediator.rs:578`、`src/rendezvous_mediator.rs:586`、`src/rendezvous_mediator.rs:650`、`src/rendezvous_mediator.rs:665`、`src/rendezvous_mediator.rs:692`、`src/rendezvous_mediator.rs:698`。
- 协议表面：`libs/hbb_common/protos/rendezvous.proto` 在相关消息中使用单个 `bytes socket_addr_v6` 字段，不是 candidate list。证据：`libs/hbb_common/protos/rendezvous.proto:20`、`libs/hbb_common/protos/rendezvous.proto:30`、`libs/hbb_common/protos/rendezvous.proto:117`、`libs/hbb_common/protos/rendezvous.proto:136`、`libs/hbb_common/protos/rendezvous.proto:157`、`libs/hbb_common/protos/rendezvous.proto:168`。
- 接口枚举先例：仓库已依赖 `default-net`，`src/lan.rs` 已使用 `default_net::get_interfaces()` 枚举接口，同时备注了 iOS simulator 构建 caveat。证据：`Cargo.toml:69`、`src/lan.rs:87`、`src/lan.rs:125`、`src/lan.rs:168`。
- 测试 / 构建：CI 使用 `cargo test --locked`；构建脚本在 Flutter packaging 前调用 `cargo build --locked`。证据：`.github/workflows/ci.yml:246`、`.github/workflows/ci.yml:250`、`build.py:319`、`build.py:321`、`build.py:405`、`build.py:409`。
- 截图 / 运行时工具：本 MVP 不改 UI，截图验收为 N/A；运行时验证通过 RustDesk 双端日志、OS socket 观察、macOS 发起端连接 Windows 受控端完成。Windows 端必须运行同一 fork commit 的构建产物；上游 stock RustDesk 只能作为对照 baseline。证据：现有 IPv6 P2P 设置位于 `flutter/lib/desktop/pages/desktop_setting_page.dart:574`、`flutter/lib/mobile/pages/settings_page.dart:800`；Windows Flutter 构建入口见 `build.py:440`、`build.py:447`，CI Windows portable 构建调用见 `.github/workflows/flutter-build.yml:256`、`.github/workflows/flutter-build.yml:258`。
- Design SSOT: `docs/design-system/design-repo.md`。
- Project Map: `docs/project-map/README.md`。
- 受影响能力：`CAP-1` 远程 P2P 连接建立。
- 验证引用：`docs/project-map/verification.md#cap-1`。

## 需求

### REQ-1 发现接口 IPv6 候选

**用户故事：** 作为使用自建 ID server 的 RustDesk 用户，我希望 RustDesk 在 STUN IPv6 解析或路由失败时仍能准备可用的全局 IPv6 P2P 候选，以便两端具备公网 IPv6 时优先尝试低延迟直连。

#### 验收标准（EARS）

1. REQ-1.1 WHEN `enable-ipv6-punch` 已启用且现有 STUN / connected-bind IPv6 路径没有产生可用地址 THEN the system SHALL 枚举本机网络接口中的 IPv6 地址作为候选来源。
2. REQ-1.2 IF 枚举到的 IPv6 地址是 loopback、unspecified、multicast、unique-local、link-local 或不在 `2000::/3` THEN the system SHALL 拒绝该地址作为公网 P2P 候选。
3. REQ-1.3 WHEN 同时存在多个有效全局 IPv6 地址 THEN the system SHALL 使用确定性的选择规则选择一个候选并记录选择来源。
4. REQ-1.4 IF 目标平台是 iOS 或现有接口枚举依赖存在构建风险的平台 THEN the system SHALL 保持当前 STUN-only 行为。

### REQ-2 验证并发布单个 IPv6 候选

**用户故事：** 作为 RustDesk 客户端，我希望接口枚举得到的 IPv6 候选在发送给对端前先被证明可绑定，以便无效地址不会降低连接可靠性。

#### 验收标准（EARS）

1. REQ-2.1 WHEN the system 选择了接口来源的 IPv6 候选 THEN the system SHALL 使用端口 `0` 绑定 UDP socket，并通过 `socket.local_addr()` 得到真实端口后编码到现有 `socket_addr_v6` bytes。
2. REQ-2.2 IF 绑定候选地址失败 THEN the system SHALL 拒绝该候选、记录失败原因，并尝试下一个有效候选。
3. REQ-2.3 IF 没有任何接口来源候选可绑定 THEN the system SHALL 保持 `socket_addr_v6` 为空，并继续现有 IPv4 UDP、TCP punch 和 relay fallback。

### REQ-3 保持现有 P2P 语义

**用户故事：** 作为维护者，我希望 MVP 复用 RustDesk 当前单候选 IPv6 rendezvous 路径，以便不需要同步修改 rustdesk-server 协议即可验证收益。

#### 验收标准（EARS）

1. REQ-3.1 WHEN 本地 IPv6 候选可用 THEN the system SHALL 通过现有 `PunchHoleRequest.socket_addr_v6` 字段发送。
2. REQ-3.2 WHEN 受控端收到 peer `socket_addr_v6` THEN the system SHALL 复用现有 `start_ipv6()` responder 路径，并通过现有 response 消息返回本端 `socket_addr_v6`。
3. REQ-3.3 IF peer 未返回有效 IPv6 socket address 或 IPv6 UDP 尝试失败 THEN the system SHALL 继续现有并发 UDP/TCP/relay 连接尝试。
4. REQ-3.4 WHEN STUN IPv6 discovery 成功 THEN the system SHALL 在 MVP 中保持当前 STUN-derived 行为优先。

### REQ-4 尊重现有配置边界

**用户故事：** 作为使用自建 ID server 的 operator，我希望新行为仍受 RustDesk 现有 IPv6 P2P 开关控制，以便连接策略可预测。

#### 验收标准（EARS）

1. REQ-4.1 WHEN `enable-ipv6-punch` 已禁用 THEN the system SHALL 不枚举接口 IPv6 候选，也不发布 `socket_addr_v6`。
2. REQ-4.2 WHEN 使用自建 rendezvous server 且 `enable-ipv6-punch` 没有被显式启用 THEN the system SHALL 保留当前禁用 IPv6 punch 的默认行为。
3. REQ-4.3 WHEN `enable-ipv6-punch` 被显式启用 THEN the system SHALL 允许接口来源 fallback，不因 rendezvous server 是自建服务器而禁用。
4. REQ-4.4 IF 需要新增可见 UI 开关 THEN the system SHALL 先回到 PRD 修改范围，再进入设计或实现。

### REQ-5 提供可诊断日志

**用户故事：** 作为诊断 relay fallback 的 maintainer，我希望日志能区分 STUN 成功、STUN 失败、接口 fallback 成功和没有 IPv6 候选，以便判断 RustDesk 是否准备进入 IPv6 P2P。

#### 验收标准（EARS）

1. REQ-5.1 WHEN STUN IPv6 discovery 失败且接口 fallback 开始 THEN the system SHALL 输出标识 fallback 路径的日志。
2. REQ-5.2 WHEN 接口 IPv6 地址被拒绝 THEN the system SHALL 记录简短拒绝原因，且不输出无关接口元数据。
3. REQ-5.3 WHEN 可绑定的接口来源 IPv6 候选被选中 THEN the system SHALL 记录候选来源为本机接口发现。
4. REQ-5.4 IF 没有 IPv6 候选可用 THEN the system SHALL 在正常 fallback 继续前记录 IPv6 P2P 候选准备失败。

### REQ-6 控制隐私和兼容性风险

**用户故事：** 作为 RustDesk 用户，我希望只有启用 IPv6 P2P 时才暴露最小必要网络地址信息，以便直连优化不扩大地址共享范围。

#### 验收标准（EARS）

1. REQ-6.1 WHEN IPv6 P2P 已禁用 THEN the system SHALL 不向 rendezvous 路径发送任何接口来源 IPv6 地址。
2. REQ-6.2 WHEN IPv6 P2P 已启用 THEN the system SHALL 在 MVP 中最多发送一个编码后的 IPv6 socket address，因为当前协议只有单个 `socket_addr_v6` 字段。
3. REQ-6.3 IF 后续添加多个 IPv6 candidate THEN the system SHALL 先完成协议和 server 兼容性设计，再实现。

### REQ-7 支持多设备 E2E 验收

**用户故事：** 作为维护者，我希望 macOS 发起端和 Windows 受控端都运行同一 fork build，并以真实连接类型、IPv6 连通性、UDP 打洞结果和延迟指标作为验收依据，以便确认功能不是只在单机或日志层面成立。

#### 验收标准（EARS）

1. REQ-7.1 WHEN 执行本 feature 的多设备 E2E THEN the system SHALL 要求 macOS 和 Windows 两端运行同一个 fork commit 的构建产物，并记录 commit hash、artifact 路径和启动时间。
2. REQ-7.2 IF Windows 设备没有本地构建环境 THEN the system SHALL 允许 Windows 直接运行从 CI 或另一台机器生成的同 commit Windows artifact；上游 stock RustDesk SHALL 只作为 baseline，不作为 feature 验收端。
3. REQ-7.3 WHEN E2E 处于可 IPv6 直连拓扑 THEN the system SHALL 以最终连接类型 `IPv6` 作为硬性通过条件，并将 `Relay`、`WebSocket` 或仅 TCP punch 视为 direct-path 验收失败。
4. REQ-7.4 WHEN E2E 建立 IPv6 direct 连接 THEN the system SHALL 记录连接建立耗时、会话延迟样本、候选来源、是否出现 relay request，并与同一拓扑 forced relay baseline 对比。
5. REQ-7.5 IF IPv6 direct 会话延迟中位数不低于 forced relay baseline THEN the system SHALL 将性能验收标为失败或 blocked，并要求记录路由、防火墙、运营商路径或候选选择原因。
6. REQ-7.6 WHEN STUN IPv6 DNS / route 在 macOS 侧被 Clash Party 或 mihomo 阻断 THEN the system SHALL 仍通过接口 fallback 准备非空 `socket_addr_v6`，并在 Windows 受控端回传本端 `socket_addr_v6`。

## 推荐范围

MVP:

- 在 `src/common.rs` 添加可单测的 IPv6 地址分类函数。
- 在 `test_ipv6()` 的 STUN / connected-bind 路径失败后加入接口枚举 fallback。
- 优先复用现有 `default-net` 依赖，不引入 `if_addrs`。
- 复用 `PUBLIC_IPV6_ADDR`、`get_ipv6_socket()` 和现有 `socket_addr_v6` 消息字段。
- 复用现有 `enable-ipv6-punch` 开关，不新增可见 UI。

后续阶段:

- 多 candidate protobuf 字段、rustdesk-server 转发、客户端 candidate racing。
- 如果需要用户可见诊断，再单独设计设置页或诊断页。

## 非目标

- MVP 不实现 full ICE、TURN、多 candidate racing。
- MVP 不修改 rustdesk-server，不要求自建 hbbs/hbbr 协议升级。
- MVP 不保证 IPv6 被防火墙、路由策略、隐私地址策略阻断时仍能 P2P 成功。
- MVP 不逆向 UU Remote 内部实现，只修复 RustDesk 当前代码证据确认的 IPv6 候选准备缺口。

## 验收定义

### 代码级

- REQ-1.1、REQ-1.2、REQ-1.3：单元测试 SHALL 覆盖 loopback、unspecified、multicast、unique-local、link-local、全局稳定地址、全局临时地址形态的过滤和确定性选择。
- REQ-1.4：目标平台 gating SHALL 避免 iOS 构建路径调用 `default_net::get_interfaces()`。
- REQ-2.1、REQ-2.2、REQ-2.3：测试 SHALL 覆盖候选绑定成功、绑定失败、无候选 fallback。
- REQ-3.1、REQ-3.2、REQ-3.3：客户端和 rendezvous mediator SHALL 继续编译并使用现有 `socket_addr_v6` 字段。
- REQ-4.1、REQ-4.2、REQ-4.3：测试 SHALL 验证接口 fallback 受 `get_ipv6_punch_enabled()` 控制，且不改变自建 server 空配置默认禁用行为。
- REQ-5.1、REQ-5.2、REQ-5.3、REQ-5.4：日志或测试 hook SHALL 区分每种发现结果。
- REQ-6.1、REQ-6.2：测试 SHALL 验证 MVP 只发布零个或一个 IPv6 socket address。
- REQ-7.1、REQ-7.3、REQ-7.4：日志 SHALL 能在双端 E2E 中识别 fork commit、候选来源、最终连接类型和连接建立耗时；如果现有日志不足，implementation SHALL 添加最小诊断日志。

### 多设备 E2E / 浏览器交互

- 浏览器交互：N/A，本功能不经过浏览器页面。
- 设备要求：macOS 发起端和 Windows 受控端 SHALL 运行同一个 fork commit 的构建产物。Windows 设备不必须 clone 仓库；它可以运行本机 clone 编译得到的产物，也可以运行 CI / 另一台机器生成的同 commit artifact。上游 stock RustDesk 只允许作为 baseline。
- Windows 操作：Windows 设备 SHALL 配置同一自建 ID server / relay server / key，启用 `enable-ipv6-punch` 与 `enable-udp-punch`，关闭 force relay / proxy / WebSocket-only 路径，并允许 RustDesk UDP 入站/出站通过 Windows 防火墙。
- IPv6 预检：双端 SHALL 记录全局 IPv6 地址；优先执行 `ping -6` 或等价命令确认基础 IPv6 连通。如果 ICMP 被阻断，测试 SHALL 记录为 `ICMP blocked`，但仍必须通过 RustDesk 日志和 UDP socket 证据继续验证。
- STUN 受阻用例：在 macOS 有全局 IPv6 且 `enable-ipv6-punch` 开启时，当 Clash Party 或 mihomo 造成 STUN IPv6 DNS / route 失败，RustDesk SHALL 记录 interface fallback，并在发起连接前准备非空 `socket_addr_v6`。
- Direct path 用例：macOS 连接 Windows，双方有可用全局 IPv6 时，RustDesk SHALL 最终建立 `IPv6` 类型连接；出现 `Relay`、`WebSocket` 或仅 TCP punch 时，该 direct-path E2E SHALL 不通过。
- 延迟用例：同一网络拓扑下 SHALL 先记录 forced relay baseline，再记录 IPv6 direct 的连接建立耗时和会话延迟样本；IPv6 direct 的会话延迟中位数 SHALL 低于 forced relay baseline，否则标为性能失败或 blocked。
- 回退用例：当对端无可用 IPv6、UDP 被防火墙阻断或候选为空时，RustDesk SHALL 回退到现有 IPv4 UDP / TCP punch / relay 流程，并在日志中标明 direct-path 未通过的原因。
- 重复次数：direct path 用例 SHALL 连续运行至少 5 次，forced relay baseline SHALL 至少运行 3 次；每次记录连接类型、候选来源、连接建立耗时、会话延迟中位数和是否出现 relay request。

### 截图 / 设计回归

N/A，本 MVP 不新增 UI，也不修改设置页现有 IPv6 P2P 开关。若范围改为新增诊断 UI，必须先更新 PRD 并进入 Phase 2 设计。

### 项目地图验证引用

- `CAP-1`：`docs/project-map/verification.md#cap-1`。

## NFR

- 可靠性：IPv6 候选准备或连接失败时，relay fallback 必须保持可用。
- 隐私：只有开启 IPv6 P2P 时才发送最多一个接口来源全局 IPv6 socket address。
- 兼容性：MVP 保持 wire compatibility，不修改 rustdesk-server 协议。
- 可观测性：日志必须区分 STUN 成功、接口 fallback 成功和没有 IPv6 候选。

## 冻结建议

建议冻结为：当前阶段只做单候选接口 IPv6 fallback；不新增 UI；不改 rustdesk-server；多 candidate 协议作为后续 feature。
