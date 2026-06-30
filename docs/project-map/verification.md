# 验证规格

## CAP-1 远程 P2P 连接建立

verification_ref: `verification.md#cap-1`

### 验证类型

- `static`
- `integration`
- `runtime`
- `functional`

### 运行入口

- 静态与集成入口：`cargo test --locked --lib`、`cargo build --locked --lib`。
- 运行时入口：macOS RustDesk 客户端连接 Windows RustDesk 客户端，自建 ID server 可用，两端客户端显式启用 `enable-ipv6-punch` 和 `enable-udp-punch`。
- 双端版本要求：macOS 发起端和 Windows 受控端必须运行同一个 fork commit 的构建产物。Windows 设备可以本机 clone + build，也可以直接运行 CI / 另一台机器产出的该 fork Windows artifact；上游 stock RustDesk 只能作为对照 baseline，不能作为本 feature 的验收端。
- 截图入口：N/A，当前能力变更为核心网络行为，不改变 UI。

### 多设备 E2E 步骤

浏览器步骤：N/A，当前能力通过桌面客户端运行时日志、系统 socket 状态、双端连接行为和延迟指标验证，不通过浏览器页面交互验证。

1. 构建或获取同一个 fork commit 的 macOS 和 Windows 客户端产物；在两端记录 commit hash、构建时间和 artifact 路径。
2. Windows 设备安装或运行该 fork 构建产物，配置与 macOS 相同的自建 ID server / relay server / key，启用无人值守或本次连接所需授权，确保 Windows 防火墙允许 RustDesk 入站和出站 UDP。
3. macOS 设备运行同一 fork commit 的构建产物，配置同一自建 ID server / relay server / key，启用 `enable-ipv6-punch` 和 `enable-udp-punch`，关闭 force relay / proxy / WebSocket-only 路径。
4. 双端预检 IPv6：记录各自全局 IPv6 地址；优先执行 `ping -6` 或平台等价命令验证基础 IPv6 连通。如果 ICMP 被运营商或防火墙阻断，则记录为“ICMP blocked”，但必须继续通过 RustDesk 日志和 UDP socket 证据验证。
5. STUN 受阻场景：在 macOS 侧使用 Clash Party 或 mihomo 让 STUN IPv6 DNS / route 失败，同时保持普通 IPv6 出站可用；确认日志出现 STUN IPv6 失败和 interface fallback 开始。
6. 从 macOS 发起连接到 Windows，至少连续执行 5 次连接尝试；每次记录最终连接类型、候选来源、是否出现 relay request、连接建立耗时、会话延迟样本。
7. forced relay baseline：在同一网络拓扑下强制 relay 或禁用 direct path，执行至少 3 次连接，记录 relay 连接建立耗时和会话延迟样本。
8. 可选反向验证：从 Windows 发起连接到 macOS，重复候选准备、连接类型和延迟记录，用于确认双端角色互换后仍可工作。

### 涉及模块

- Rust 核心网络
- 发起端连接流程
- 受控端 rendezvous mediator
- rendezvous protobuf
- Flutter 设置页

### 关键断言

- 当 `enable-ipv6-punch` 关闭时，客户端不准备或发送 IPv6 P2P 候选。
- 当 `enable-ipv6-punch` 开启且本机存在可绑定的全局 IPv6 地址时，客户端在连接前准备非空 `socket_addr_v6`。
- 当 STUN IPv6 DNS 或路由失败但本机存在可用全局 IPv6 地址时，客户端仍尝试接口枚举 fallback。
- 在可直连 IPv6 拓扑下，最终连接类型必须为 `IPv6`，不能是 `Relay`、`WebSocket` 或仅 TCP punch。
- IPv6 direct 的会话延迟中位数必须低于同一拓扑 forced relay baseline；若不低于 baseline，本次 E2E 标记为性能未通过并需要原因分析。
- 每次 E2E 必须记录连接建立耗时，优先使用 RustDesk 日志中的 `used to establish IPv6 connection` / `used to establish Relay connection`。
- 当 IPv6 UDP 连接失败时，现有 IPv4 UDP、TCP punch、relay 回退路径仍可继续。
- 当自建 ID server 被使用时，客户端设置必须显式启用 IPv6 P2P，默认策略不被本功能静默放宽。

### 边界场景

- 本机只有 loopback、link-local、unique-local、multicast 或 unspecified IPv6 地址。
- 本机同时存在稳定 IPv6 地址和临时隐私 IPv6 地址。
- Clash Party 或 mihomo TUN 导致 STUN IPv6 解析失败。
- 对端不返回 `socket_addr_v6`。
- 对端返回 IPv6 地址但本机防火墙或路由阻断 UDP。

### 清理要求

- 运行时验证结束后恢复 Clash Party 或 mihomo 配置到用户原状态。
- 保留验证日志路径和关键时间点，不持久化敏感服务器 key、token 或完整公网地址清单。
- Windows 设备恢复原 RustDesk 安装或服务状态；如果使用 portable artifact，删除临时目录和临时日志。
