# Implementation Log

- 2026-06-30: 克隆最新 `rustdesk/rustdesk`，初始化 `libs/hbb_common`，完成 interface IPv6 candidate P2P 的 Phase 1 代码调研和 PRD 草稿。
- 2026-06-30: 按新版 workflow 补齐最小 `docs/design-system/` 与 `docs/project-map/`，并将 Phase 1 三件套同步为中文规范结构。
- 2026-06-30: 根据反馈补强多设备 E2E：明确 Windows 端必须运行同 commit fork artifact，增加 IPv6 连通、UDP 打洞、连接类型和延迟 baseline 验收。
- 2026-06-30: 用户确认冻结 PRD；新增 `development-pipeline.md` 和 `implementation-goal.md`，明确 Mac 验证、fork 推送、Windows 同 commit E2E 和真正代码落地目标。
- 2026-06-30: 完成 Phase 2 最小设计交接草案：`design.md` 和 `design-handoff.md` 定义无 UI / 无 motion / 无截图的核心网络改动方案，等待用户批准进入任务拆分。
- 2026-06-30: 用户批准 Phase 2，`design.md` 与 `design-handoff.md` 标记为 frozen，进入 Phase 3 任务拆分。
- 2026-06-30: 生成 Phase 3 任务计划：`tasks/IMPL-001.md` 到 `tasks/IMPL-004.md`，并生成 `traceability-matrix.md` 覆盖 REQ-1 到 REQ-7，等待用户批准后进入源码实现。
- 2026-06-30: Phase 3 自检通过：四个 IMPL 文件均包含范围、EARS、验证命令、runtime 标记和接线检查；`traceability-matrix.md` 覆盖 REQ-1.1 到 REQ-7.6。当前仍等待任务计划批准；本地 remote 仅有 upstream `origin`，feature branch 和 `neko` remote 留到批准后执行。
- 2026-06-30: 用户批准 Phase 3 任务计划；创建分支 `feat/interface-ipv6-candidate-p2p`，添加 remote `neko=https://github.com/NEKO-CwC/rustdesk.git`，进入 Phase 4 并开始 IMPL-001。
- 2026-06-30: 完成代码第一版：在 `src/common.rs` 增加接口 IPv6 候选过滤、确定性排序、非 iOS 接口枚举、bind fallback 和单元测试；在 `src/client.rs` / `src/rendezvous_mediator.rs` 增加 `socket_addr_v6` 非空诊断日志。`cargo test --locked --lib ipv6_candidate` 首次因 git 子模块网络失败中断，重试后进入编译但被 macOS 原生依赖阻塞：缺少 `libyuv` / vcpkg 依赖；尝试 Homebrew 安装时发现 Homebrew 无 `libyuv` formula 且安装 `nasm yasm` 被 `/usr/local/share/man/man8` root 权限阻塞。已按 README clone 并 bootstrap `/Users/neko/vcpkg`，但 vcpkg install 还需 `nasm`。
- 2026-06-30: 通过本地提取 Homebrew bottle 的 `nasm` / `yasm` 绕过 `/usr/local/share/man/man8` root 权限问题，并使用 `/Users/neko/vcpkg` 完成 `libvpx`、`libyuv`、`opus`、`aom` 安装。
- 2026-06-30: 收紧 IPv6 候选分类，拒绝 documentation 前缀 `2001:db8::/32`；补充 connected-bind 返回不可发布地址时进入 interface fallback 的分支和单测。
- 2026-06-30: macOS 代码级验证通过：`ipv6_candidate` 3 个测试、`interface_ipv6` 4 个测试、`ipv6` 11 个相关测试、全量 `cargo test --locked --lib` 73 个测试、`cargo build --locked --lib` 均通过；日志 grep 和 protobuf / Flutter 无 diff 检查通过。IMPL-001/002/003 标记 complete，IMPL-004 保持 in_progress，等待提交、推送 fork 与 Windows 同 commit E2E。
- 2026-07-01: Mac -> Windows 同 commit 实测仍为 relay：Mac 日志显示已准备非空 `socket_addr_v6`，Windows 日志显示 `create_relay requested` 且没有 `Prepared non-empty socket_addr_v6 for IPv6 responder`。结论：旧 hbbs 未转发 `socket_addr_v6`，客户端单独修改不足以完成 P2P。
- 2026-07-01: 在 `/Users/neko/Documents/Project/rustdesk-server` 增加 patched hbbs：server proto 对齐客户端连接字段，`rendezvous_server.rs` 转发 `socket_addr_v6`、UDP 和 UPnP 字段；`cargo check --locked` 与 `cargo build --release --locked --bins` 通过。本机 release 产物为 macOS arm64，不能直接部署 Linux server；真实 E2E 前必须在实际 ID server 上部署 patched hbbs。
