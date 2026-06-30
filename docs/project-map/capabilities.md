# 能力地图

```yaml
- id: CAP-1
  name: 远程 P2P 连接建立
  roles: [user, operator, maintainer]
  depends_on_modules:
    - Rust 核心网络
    - 发起端连接流程
    - 受控端 rendezvous mediator
    - rendezvous protobuf
    - Flutter 设置页
    - 多设备运行时验证
  journey_nodes: [CAP-1]
  status: complete
  last_verified: repo-snapshot-2026-06-30
  verification_ref: verification.md#cap-1
```

## 模块说明

- Rust 核心网络：IPv4/IPv6 探测、STUN、UDP socket 绑定、候选地址缓存。
- 发起端连接流程：发起连接、发送 `PunchHoleRequest`、并发尝试 IPv6 UDP、IPv4 UDP、TCP punch 和 relay。
- 受控端 rendezvous mediator：处理 punch、intranet、relay response，并在受控端启动 IPv6 UDP 监听。
- rendezvous protobuf：客户端和服务端之间传递 `socket_addr_v6` 的消息字段。
- Flutter 设置页：暴露 `enable-ipv6-punch` 与 `enable-udp-punch` 等本地连接策略开关。
- 多设备运行时验证：macOS 和 Windows 双端运行同一 fork commit 构建产物，验证 IPv6 连通、UDP 打洞、连接类型和延迟指标。
