# Implementation Handoff - Interface IPv6 Candidate P2P

Status: client_verified_server_deploy_pending

## 当前状态

macOS 本机代码级实现已完成并验证通过，客户端 fork 已推送。2026-07-01 的 Mac -> Windows 实测显示当前会话仍为 relay；根因证据指向旧 hbbs 未转发 `socket_addr_v6`，所以真实 E2E 的下一步是部署 patched `rustdesk-server`，不是继续只换客户端。

当前分支：

```text
feat/interface-ipv6-candidate-p2p
```

当前 remote：

```text
neko=https://github.com/NEKO-CwC/rustdesk.git
```

提交状态：

- 推送后，以本文件所在 commit 为准。
- Windows 端必须运行 `git rev-parse HEAD` 得到的同一 commit，不能使用上游 stock RustDesk 作为 feature 验收端。

## 已完成改动

- `src/common.rs`
  - 新增 IPv6 候选分类：拒绝 loopback、unspecified、multicast、unique-local、link-local、documentation `2001:db8::/32` 和非 `2000::/3` 地址。
  - 非 iOS 平台通过 `default_net::get_interfaces()` 枚举接口 IPv6，iOS 保持空候选。
  - 多候选使用数值排序和去重，避免依赖 OS 接口枚举顺序。
  - 在 `test_bind_ipv6()` 失败，或 connected-bind 返回不可发布地址时，进入 interface fallback。
  - fallback 对候选绑定 UDP port `0`，成功后把地址以 port `0` 缓存给现有 `get_ipv6_socket()`，由后者重新绑定真实端口并编码 `socket_addr_v6`。
  - STUN async discovery 保持运行；如果 STUN 成功，仍可覆盖缓存，保持 STUN-derived 优先。
- `src/client.rs`
  - 发起端补充 `socket_addr_v6` 是否非空的诊断日志。
- `src/rendezvous_mediator.rs`
  - 受控端 IPv6 responder 补充 `socket_addr_v6` 是否非空的诊断日志。

## Server 前置修正

本地 server 补丁仓库：

```text
/Users/neko/Documents/Project/rustdesk-server
```

已完成并通过：

```bash
cargo check --locked
cargo build --release --locked --bins
```

补丁内容：

- `libs/hbb_common/protos/rendezvous.proto` 对齐客户端连接相关字段，包含 IPv6、UDP、UPnP、relay response 和 local addr 相关字段。
- `src/rendezvous_server.rs` 转发 `socket_addr_v6`：`PunchHoleRequest -> PunchHole/FetchLocalAddr`，`PunchHoleSent/LocalAddr -> PunchHoleResponse`，`RelayResponse -> requester`。
- hbbs debug log 记录 `socket_addr_v6_non_empty`，避免泄漏完整地址。

注意：

- 本机 release 产物是 macOS arm64：`target/release/hbbs`、`target/release/hbbr`，不能直接部署到 Linux server。
- `rustdesk-server` 工作区里 `db_v2.sqlite3` 是本地运行产生的未提交脏文件，不属于补丁。

## 验证结果

以下命令均在 macOS 本机通过。为了满足 RustDesk macOS native 依赖，命令使用了本机 vcpkg 与本地提取的 `nasm` / `yasm`。

```bash
VCPKG_ROOT=/Users/neko/vcpkg PATH=/Users/neko/.local/homebrew-tools/nasm/3.02/bin:/Users/neko/.local/homebrew-tools/yasm/1.3.0_2/bin:$PATH cargo test --locked --lib ipv6_candidate
```

结果：通过，3 个候选分类测试通过。

```bash
VCPKG_ROOT=/Users/neko/vcpkg PATH=/Users/neko/.local/homebrew-tools/nasm/3.02/bin:/Users/neko/.local/homebrew-tools/yasm/1.3.0_2/bin:$PATH cargo test --locked --lib interface_ipv6
```

结果：通过，4 个 interface fallback bind 测试通过。

```bash
VCPKG_ROOT=/Users/neko/vcpkg PATH=/Users/neko/.local/homebrew-tools/nasm/3.02/bin:/Users/neko/.local/homebrew-tools/yasm/1.3.0_2/bin:$PATH cargo test --locked --lib ipv6
```

结果：通过，11 个 IPv6 相关测试通过。

```bash
VCPKG_ROOT=/Users/neko/vcpkg PATH=/Users/neko/.local/homebrew-tools/nasm/3.02/bin:/Users/neko/.local/homebrew-tools/yasm/1.3.0_2/bin:$PATH cargo test --locked --lib
```

结果：通过，73 个 lib 测试通过。

```bash
VCPKG_ROOT=/Users/neko/vcpkg PATH=/Users/neko/.local/homebrew-tools/nasm/3.02/bin:/Users/neko/.local/homebrew-tools/yasm/1.3.0_2/bin:$PATH cargo build --locked --lib
```

结果：通过。

```bash
rg "interface fallback|socket_addr_v6|IPv6 candidate|used to establish" src/common.rs src/client.rs src/rendezvous_mediator.rs
git diff -- libs/hbb_common/protos/rendezvous.proto flutter
```

结果：日志路径可定位；protobuf 与 Flutter UI 无 diff。

备注：测试和构建输出包含大量 RustDesk 上游既有 macOS / objc / deprecated warning，本 feature 没有把 warning 当作通过条件。

## 推送前步骤

1. 查看 diff：

```bash
git diff -- src/common.rs src/client.rs src/rendezvous_mediator.rs .workflow/design-implementation docs/design-system docs/project-map .gitignore
```

2. 创建 commit：

```bash
git add .gitignore src/common.rs src/client.rs src/rendezvous_mediator.rs .workflow/design-implementation docs/design-system docs/project-map
git commit -m "feat: add interface ipv6 candidate fallback"
```

3. 记录 commit hash：

```bash
git rev-parse HEAD
```

4. 推送给 Windows 端：

```bash
git push -u neko feat/interface-ipv6-candidate-p2p
```

## Windows 端执行 prompt

```text
你在 Windows 设备上验证 NEKO-CwC/rustdesk 的 feat/interface-ipv6-candidate-p2p 分支。请确保运行的 RustDesk artifact 或源码 commit 与 macOS 端完全一致。

任务：
1. 记录 commit hash、artifact 路径、启动时间。
2. 确认自建 ID server 正在运行 patched hbbs，并配置与 macOS 相同的 ID server / relay server / key。
3. 启用 enable-ipv6-punch 和 enable-udp-punch，关闭 force relay、proxy、WebSocket-only。
4. 确认 Windows 有全局 IPv6：运行 ipconfig，记录 IPv6 地址；如果可以，ping -6 macOS 的 IPv6。如果 ICMP 被阻断，记录 ICMP blocked。
5. 放行 Windows 防火墙中 RustDesk 的 UDP 入站和出站。
6. 等待 macOS 发起连接，连续测试 5 次。
7. 每次记录最终连接类型、是否 Relay、是否出现 socket_addr_v6、候选来源、连接建立耗时、会话延迟中位数。
8. 再做 3 次 forced relay baseline，对比延迟中位数。
9. 输出表格：commit、Windows IPv6、Mac IPv6、候选来源、最终连接类型、连接耗时、延迟中位数、是否通过。
```

## 多设备 E2E 通过条件

- macOS 和 Windows 运行同一个 fork commit。
- ID server 运行 patched hbbs，且 hbbs 日志能看到 `socket_addr_v6_non_empty=true`。
- 双端均有全局 IPv6；如果 ICMP 被阻断，需要日志和 UDP socket 证据继续验证。
- macOS 在 STUN IPv6 DNS / route 受阻时仍准备非空 `socket_addr_v6`。
- Windows 受控端回传非空 `socket_addr_v6`。
- 最终连接类型必须为 `IPv6`；`Relay`、`WebSocket` 或仅 TCP punch 均不通过 direct-path 验收。
- IPv6 direct 的会话延迟中位数必须低于同拓扑 forced relay baseline。
- direct path 至少 5 次，forced relay baseline 至少 3 次。

## 未关闭风险

- 未在真实 Windows fork artifact 上验证最终连接类型。
- 未部署 patched hbbs 到实际 ID server；旧 hbbs 已证明会导致 `socket_addr_v6` 无法到达 Windows。
- 未验证 Clash Party / mihomo 阻断 STUN IPv6 时的真实日志。
- 未验证运营商 IPv6 入站、防火墙和 NAT 行为是否允许 UDP direct。
- 未验证 direct 延迟是否低于 forced relay baseline。
