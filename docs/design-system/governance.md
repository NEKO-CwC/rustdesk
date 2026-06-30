# Governance

## 状态

本设计 SSOT 当前为 `active`，范围是 RustDesk 二开仓库的最小 UI 规则集合。

## 变更规则

- LOCK：仓库级 token、motion、canonical component 的变化必须修改 `docs/design-system/`，不能只写在 per-feature `design.md`。
- LOCK：per-feature 设计只能引用 `design_repo_refs`。
- GUIDE：只有实际 UI 需求出现时，才扩展组件和页面模式。

## Token 晋升

一个裸值在三个以上 UI 场景稳定复用，并且不能由现有 semantic token 表达时，才晋升为 token。
