# Implementation Baseline - Interface IPv6 Candidate P2P

`prd.md` 当前为 `awaiting_user`。`design.md` 与 `design-handoff.md` 尚未进入，因为 Phase 1 仍在用户审批门。

## 仓库快照

- 主仓库路径：`/Users/neko/Documents/Project/rustdesk`。
- 主仓库 HEAD：`b3bd18845d7d9b7af565eabef30104bcb17810ac`。
- `libs/hbb_common` 是声明在 `libs/hbb_common` 的 git submodule，URL 为 `https://github.com/rustdesk/hbb_common`。证据：`.gitmodules:1`、`.gitmodules:2`、`.gitmodules:3`。
- `libs/hbb_common` 已初始化到主仓库记录的 `a920d00945e1d2441b3f77b2677054cb8c3d9dd2`。

## 仓库级 SSOT

- Design SSOT 已建立：`docs/design-system/design-repo.md`，状态为 `active`。
- Project Map 已建立：`docs/project-map/README.md`，状态为 `active`。
- 本 feature 影响能力：`CAP-1` 远程 P2P 连接建立。
- 验证引用：`docs/project-map/verification.md#cap-1`。

## 代码调研结论

- RustDesk 是 Rust 2021 package，名称为 `rustdesk`，版本为 `1.4.8`，library target 为 `librustdesk`。证据：`Cargo.toml:1`、`Cargo.toml:3`、`Cargo.toml:5`、`Cargo.toml:11`。
- 本功能相关核心模块是 `client`、`lan`、`rendezvous_mediator`、`common`。证据：`src/lib.rs:14`、`src/lib.rs:15`、`src/lib.rs:17`、`src/lib.rs:21`。
- 当前 IPv6 候选缓存是单个全局 slot `PUBLIC_IPV6_ADDR`，与协议单地址形态一致。证据：`src/common.rs:95`、`src/common.rs:99`。
- UDP punch 和 IPv6 punch 是已有本地 option，读取入口为 `get_udp_punch_enabled()` 和 `get_ipv6_punch_enabled()`。证据：`src/common.rs:1093`、`src/common.rs:1100`。
- 自建 rendezvous server 下，UDP/IPv6 punch option 为空时 RustDesk 返回 `"N"`，因此用户需要显式启用。证据：`src/common.rs:1107`、`src/common.rs:1109`、`src/common.rs:1111`。
- 当前 IPv6 STUN discovery 会解析公开 STUN 域名并筛选 IPv6 地址。证据：`src/common.rs:2321`、`src/common.rs:2326`、`src/common.rs:2328`、`src/common.rs:2331`。
- `test_bind_ipv6()` 同样依赖 `STUNS_V6[0]` 解析到 IPv6 地址，再通过 `socket.local_addr()` 得到本地 IPv6。证据：`src/common.rs:2401`、`src/common.rs:2404`、`src/common.rs:2406`、`src/common.rs:2414`。
- `test_ipv6()` 对 discovery 有 60 秒节流，先尝试 `test_bind_ipv6()`，再异步跑 STUN probe，失败时记录 public IPv6 获取失败。证据：`src/common.rs:2418`、`src/common.rs:2419`、`src/common.rs:2430`、`src/common.rs:2478`、`src/common.rs:2497`。
- `test_ipv6()` 内有被注释的 `if_addrs` 枚举逻辑，并注明 macOS 上本地 IPv6 查询曾不稳定。证据：`src/common.rs:2448`、`src/common.rs:2450`、`src/common.rs:2452`、`src/common.rs:2454`。
- `get_ipv6_socket()` 绑定缓存地址，读取 `socket.local_addr()` 并用 `AddrMangle::encode` 编码一个 socket address。证据：`src/common.rs:2564`、`src/common.rs:2565`、`src/common.rs:2569`、`src/common.rs:2574`、`src/common.rs:2577`。
- 发起端在 IPv6 punch 开启时调用 `test_ipv6()`，再调用 `get_ipv6_socket()`，并把结果放入 `PunchHoleRequest.socket_addr_v6`。证据：`src/client.rs:311`、`src/client.rs:451`、`src/client.rs:452`、`src/client.rs:462`、`src/client.rs:471`。
- 发起端会从 `PunchHoleResponse` 和 `RelayResponse` 消费返回的 IPv6 socket address，并通过 `udp_nat_connect()` 尝试 IPv6。证据：`src/client.rs:523`、`src/client.rs:524`、`src/client.rs:527`、`src/client.rs:536`、`src/client.rs:545`、`src/client.rs:549`。
- 受控端解码 peer `socket_addr_v6`，启动 `start_ipv6()`，并在 relay 或 punch response 中带回本端 `socket_addr_v6`。证据：`src/rendezvous_mediator.rs:578`、`src/rendezvous_mediator.rs:586`、`src/rendezvous_mediator.rs:650`、`src/rendezvous_mediator.rs:665`、`src/rendezvous_mediator.rs:692`、`src/rendezvous_mediator.rs:698`。
- relay fallback 仍由 symmetric NAT、proxy/WebSocket relay、force relay、禁用 TCP listen 且无 UDP port 等条件触发。证据：`src/rendezvous_mediator.rs:659`、`src/rendezvous_mediator.rs:671`、`src/rendezvous_mediator.rs:674`。
- `start_ipv6()` 会再次调用 `test_ipv6()`，再通过 `get_ipv6_socket()` 启动 `udp_nat_listen()`。证据：`src/rendezvous_mediator.rs:929`、`src/rendezvous_mediator.rs:935`、`src/rendezvous_mediator.rs:936`、`src/rendezvous_mediator.rs:938`、`src/rendezvous_mediator.rs:940`。
- rendezvous protobuf 在相关消息中均为单个 `bytes socket_addr_v6` 字段，不是 candidate list。证据：`libs/hbb_common/protos/rendezvous.proto:20`、`libs/hbb_common/protos/rendezvous.proto:30`、`libs/hbb_common/protos/rendezvous.proto:117`、`libs/hbb_common/protos/rendezvous.proto:136`、`libs/hbb_common/protos/rendezvous.proto:157`、`libs/hbb_common/protos/rendezvous.proto:168`。
- 仓库已依赖 `default-net`，`src/lan.rs` 已使用 `default_net::get_interfaces()`。证据：`Cargo.toml:69`、`src/lan.rs:87`、`src/lan.rs:125`。
- `src/lan.rs` 记录 `default_net::get_interfaces()` 在 iOS simulator x86_64 下有 undefined symbols 风险，因此 fallback 需要 target gate。证据：`src/lan.rs:168`、`src/lan.rs:169`。
- 桌面和移动设置页都已有 IPv6 P2P 开关。证据：`flutter/lib/desktop/pages/desktop_setting_page.dart:574`、`flutter/lib/desktop/pages/desktop_setting_page.dart:576`、`flutter/lib/mobile/pages/settings_page.dart:800`、`flutter/lib/mobile/pages/settings_page.dart:804`。
- CI 和 build script 已给出 Rust 测试与构建入口。证据：`.github/workflows/ci.yml:246`、`.github/workflows/ci.yml:250`、`build.py:319`、`build.py:321`、`build.py:405`、`build.py:409`。
- Windows Flutter 构建入口在 `build_flutter_windows()` 中调用 `cargo build --locked --features ... --lib --release` 和 `flutter build windows --release`。证据：`build.py:440`、`build.py:442`、`build.py:447`。
- CI Windows portable 构建使用 `python3 .\build.py --portable --flutter --skip-portable-pack --hwcodec ...`，可作为 Windows 验收 artifact 的来源。证据：`.github/workflows/flutter-build.yml:256`、`.github/workflows/flutter-build.yml:258`。
- 客户端已有连接类型和耗时日志：relay response 后会记录 `used to establish {typ} connection`，direct punch 后会记录 punch hole 耗时。证据：`src/client.rs:577`、`src/client.rs:596`。
- IPv6 UDP direct 连接通过 `udp_nat_connect(..., "IPv6", ...)` 进入 KCP connect，受控端通过 `udp_nat_listen()` 接受 KCP。证据：`src/client.rs:707`、`src/client.rs:708`、`src/client.rs:4286`、`src/client.rs:4297`、`src/rendezvous_mediator.rs:948`、`src/rendezvous_mediator.rs:960`。

## 确认范围

范围内：

- Rust core 中的 IPv6 候选准备。
- STUN-dependent 路径失败后的接口枚举 fallback。
- 复用现有 `enable-ipv6-punch` 和 `socket_addr_v6`。
- IPv6 地址过滤、选择、option gating 的测试。
- macOS 到 Windows 的双端 fork build E2E、日志、socket 和延迟指标验证。

范围外：

- full ICE、TURN、多 candidate racing。
- rustdesk-server 协议或转发变更。
- 新 UI 开关或诊断页。
- 逆向或复刻 UU Remote 内部实现。

## MVP 架构建议

1. 在 `src/common.rs` 抽取纯函数分类公网 IPv6 候选。
2. 添加 target-gated 的接口候选 provider，在支持平台使用 `default_net::get_interfaces()`。
3. 在 `test_ipv6()` 中保留 `test_bind_ipv6()` 优先；当它失败或无可用地址时同步尝试接口候选，使同一次连接里的 `get_ipv6_socket()` 可用。
4. 保留异步 STUN probe 作为附加发现路径。
5. 继续使用 `PUBLIC_IPV6_ADDR` 和 `get_ipv6_socket()` 绑定并编码实际端口。
6. 添加日志区分 STUN、interface fallback、候选拒绝、绑定失败和无候选。

## 验证环境

- 静态与单元验证：`cargo test --locked --lib`。
- 构建验证：`cargo build --locked --lib`。
- 运行时验证：macOS RustDesk fork build + Windows RustDesk fork build，二者必须是同一个 commit；自建 ID server；Clash Party 或 mihomo 阻断 macOS 侧 STUN IPv6 DNS / route。
- Windows 设备操作：Windows 不必须 clone 仓库；可以本机 clone + build，也可以运行同 commit 的 CI / 外部构建 artifact。它必须配置同一自建 ID server / relay server / key，启用 `enable-ipv6-punch` 和 `enable-udp-punch`，并允许 RustDesk UDP 通过防火墙。
- socket 验证：RustDesk 双端日志和 OS socket 观察，确认连接前准备非空 `socket_addr_v6`，并最终建立 `IPv6` 类型连接。
- 延迟验证：同一拓扑记录 forced relay baseline 和 IPv6 direct 的连接建立耗时、会话延迟样本；IPv6 direct 会话延迟中位数应低于 forced relay baseline。
- 截图验证：N/A，本 MVP 不改 UI。

## 风险

- macOS 可能暴露稳定地址和临时隐私地址，MVP 需要确定性选择和清晰日志。
- 接口 IPv6 可绑定不代表对端一定可达，因此 relay fallback 必须保持。
- 当前协议只有一个 IPv6 地址字段，选错候选会错过其他可行地址；多候选需要后续协议设计。
- `default_net::get_interfaces()` 在 iOS 路径有现有 caveat，必须 target gate。
