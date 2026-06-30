# 开发与多设备验证流水线

Status: active

## 目标

本流水线用于把 `Interface IPv6 Candidate P2P` 从冻结 PRD 推进到代码实现、macOS 本机验证、patched hbbs 部署、推送 fork、Windows 同 commit E2E 验证。它是执行顺序说明，不替代 `prd.md`、`design.md`、`tasks/` 或 `traceability-matrix.md`。

## 阶段顺序

### 1. 冻结范围

- `prd.md` 已冻结后的客户端 MVP 范围固定为：复用现有 `enable-ipv6-punch`，复用单个 `socket_addr_v6` 字段，不新增 UI。
- 2026-07-01 runtime evidence 修正了早期假设：当前 `rustdesk-server` 会解析后重组 rendezvous 消息，旧 hbbs 不转发 `socket_addr_v6`，因此必须部署 patched hbbs 才能进行真实 IPv6 P2P 验收。
- 后续若要新增多 candidate 协议、诊断 UI 或新开关，必须重新打开 PRD。

### 2. 准备 Git 分支

当前本地仓库路径：

```bash
cd /Users/neko/Documents/Project/rustdesk
```

当前 `origin` 是上游 `https://github.com/rustdesk/rustdesk.git`。推送到个人 fork 时使用单独 remote：

```bash
git remote add neko https://github.com/NEKO-CwC/rustdesk.git
git switch -c feat/interface-ipv6-candidate-p2p
```

如果 `neko` remote 已存在，则检查 URL：

```bash
git remote get-url neko
```

预期 URL：

```text
https://github.com/NEKO-CwC/rustdesk.git
```

### 3. 提交 workflow 基线

先提交冻结 PRD、project map、design SSOT 和本流水线，让后续 Mac / Windows / CI 端都能以同一验收标准继续。

建议提交信息：

```text
docs: define ipv6 candidate p2p workflow
```

### 4. 实现顺序

代码实现按垂直切片推进：

1. 在 `src/common.rs` 添加可单测的 IPv6 地址分类和候选选择逻辑。
2. 使用现有 `default_net::get_interfaces()` 枚举本机接口地址，避免新增依赖。
3. 过滤 loopback、unspecified、multicast、unique-local、link-local 和非 `2000::/3` IPv6 地址。
4. 在 `test_ipv6()` 的 STUN / connected-bind IPv6 路径失败后进入 interface fallback。
5. 对候选 IPv6 地址绑定 UDP socket port `0`，成功后用 `socket.local_addr()` 得到真实端口。
6. 复用现有 `PUBLIC_IPV6_ADDR`、`get_ipv6_socket()` 和 `socket_addr_v6` 发布路径。
7. 增加最小诊断日志：STUN 失败、interface fallback 开始、候选被拒、候选选中、候选为空、最终连接类型。

### 5. macOS 本机验证

先执行静态与集成验证：

```bash
cargo test --locked --lib
cargo build --locked --lib
```

随后运行 macOS RustDesk fork build，验证日志中至少能看到：

- `enable-ipv6-punch` 开启时才启动 IPv6 候选准备。
- STUN IPv6 失败后进入 interface fallback。
- 至少一个接口来源全局 IPv6 候选通过 UDP bind。
- `socket_addr_v6` 在发起连接前非空。
- IPv6 候选失败时，现有 IPv4 / TCP / relay fallback 仍继续。

### 6. 推送到 fork 的时机

客户端推送发生在 macOS 代码级验证通过之后，不等待 Windows E2E 完成。原因是 Windows 必须运行同一个 fork commit，push 是跨设备同步点。

```bash
git push -u neko feat/interface-ipv6-candidate-p2p
```

推送前记录：

- branch: `feat/interface-ipv6-candidate-p2p`
- commit hash
- macOS 测试命令与结果
- macOS artifact 路径或启动方式

### 6.5. 部署 patched hbbs 前置条件

真实 E2E 前必须先部署 patched `rustdesk-server`，否则 Windows 收不到 Mac 发出的 `socket_addr_v6`，不会启动 IPv6 responder，连接会继续落到 relay。

本地 server 补丁仓库：

```bash
cd /Users/neko/Documents/Project/rustdesk-server
```

必须包含：

- `libs/hbb_common/protos/rendezvous.proto` 对齐客户端连接相关字段，至少包含 `PunchHoleRequest.socket_addr_v6`、`PunchHole.socket_addr_v6`、`PunchHoleSent.socket_addr_v6`、`PunchHoleResponse.socket_addr_v6`、`RelayResponse.socket_addr_v6`、`FetchLocalAddr.socket_addr_v6`、`LocalAddr.socket_addr_v6`。
- `src/rendezvous_server.rs` 在 A->B 和 B->A 两个方向转发 `socket_addr_v6`，并保留 UDP / UPnP 相关字段。
- `cargo check --locked` 通过。
- 目标服务器实际运行 patched `hbbs`，不是只编译客户端。

当前 macOS 本机 release 产物仅用于本机参考：

```text
/Users/neko/Documents/Project/rustdesk-server/target/release/hbbs
/Users/neko/Documents/Project/rustdesk-server/target/release/hbbr
```

注意：上述二进制是 `Mach-O arm64`，不能直接部署到常见 Linux 服务器。Linux 服务器应在服务器上构建，或使用匹配目标架构的交叉编译产物。

### 7. Windows 获取同 commit 构建

优先路径：Windows 直接运行 CI 或另一台机器生成的同 commit artifact。

备用路径：Windows 本机 clone + build。

```powershell
git clone --recursive https://github.com/NEKO-CwC/rustdesk.git
cd rustdesk
git checkout feat/interface-ipv6-candidate-p2p
git submodule update --init --recursive
python build.py --portable --flutter --skip-portable-pack --hwcodec
```

Windows 不必须使用本地源码构建；但它必须运行与 macOS 端相同 commit 的 fork artifact。上游 stock RustDesk 只能作为 baseline，不能作为本 feature 验收端。

### 8. Windows 端继续测试 prompt

```text
你在 Windows 设备上验证 NEKO-CwC/rustdesk 的 feat/interface-ipv6-candidate-p2p 分支。请确保运行的 RustDesk artifact 或源码 commit 与 macOS 端完全一致。

任务：
1. 记录 commit hash、artifact 路径、启动时间。
2. 确认自建 ID server 正在运行 patched hbbs，并配置与 macOS 相同的 ID server / relay server / key。
3. 启用 enable-ipv6-punch 和 enable-udp-punch，关闭 force relay、proxy、WebSocket-only。
4. 确认 Windows 有全局 IPv6：运行 ipconfig，记录 IPv6 地址；如果可以，ping -6 macOS 的 IPv6。
5. 放行 Windows 防火墙中 RustDesk 的 UDP 入站和出站。
6. 等待 macOS 发起连接，连续测试 5 次。
7. 每次记录最终连接类型、是否 Relay、是否出现 socket_addr_v6、连接建立耗时、会话延迟。
8. 再做 3 次 forced relay baseline，对比延迟中位数。
9. 输出表格：commit、Windows IPv6、Mac IPv6、候选来源、最终连接类型、连接耗时、延迟中位数、是否通过。
```

### 9. 多设备 E2E 通过标准

- macOS 和 Windows 运行同一个 fork commit。
- ID server 运行 patched hbbs，且 hbbs 日志能看到 `socket_addr_v6_non_empty=true`。
- 双端均记录全局 IPv6；如果 ICMP 被阻断，必须记录为 `ICMP blocked`，并继续检查 RustDesk 运行时证据。
- macOS 在 STUN IPv6 DNS / route 受阻时仍准备非空 `socket_addr_v6`。
- Windows 受控端回传非空 `socket_addr_v6`。
- 最终连接类型必须为 `IPv6`；`Relay`、`WebSocket` 或仅 TCP punch 均不算 direct-path 通过。
- IPv6 direct 的会话延迟中位数必须低于同拓扑 forced relay baseline。
- 至少完成 5 次 direct path 尝试和 3 次 forced relay baseline。

### 10. 失败分流

- 如果最终是 `Relay`：先检查 hbbs 是否已部署 patched 版本，再检查 hbbs 是否记录 `socket_addr_v6_non_empty=true`；随后检查 macOS 或 Windows 是否有一端 `socket_addr_v6` 为空，再检查 force relay / proxy / firewall / 无全局 IPv6。
- 如果 `socket_addr_v6` 非空但 IPv6 连接失败：检查 Windows 防火墙、运营商 IPv6 入站策略、UDP 端口绑定、RustDesk 日志中的 IPv6 connect 错误。
- 如果最终是 `IPv6` 但延迟仍接近 relay：记录 `ping -6`、路由路径、运营商跨网情况和 RustDesk 会话延迟样本，将性能验收标记为 failed 或 blocked。
