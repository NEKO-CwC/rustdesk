# Conformance Acceptance

## Gate 1 - 组件复用 / 可维护性

- UI-bearing work 必须复用 `components/` 中的 canonical component 或 Flutter 原生标准控件。
- 新组件必须说明为什么不能组合已有组件。

## Gate 2 - Token 正确性

- feature 设计不得重定义颜色、字体、圆角、motion 原语。
- 代码实现优先使用 `MyTheme`、`Theme.of(context)`、`ColorThemeExtension` 或 semantic token。

## Gate 3 - Motion

- 有 motion 的 feature 必须声明 `MOTION-Mx`，并提供 temporal evidence。
- 无 UI 变更的核心网络 feature 使用 `MOTION-M1` 或在 per-feature 设计中写 `Motion: N/A`。

## Gate 4 - Design QA

- 视觉质量：符合 color、typography、spacing、radius。
- 状态和变体：default、hover、pressed、disabled、error、loading 可验证。
- 响应式：desktop/mobile 影响明确。
- 内容韧性：长翻译不溢出。
- 可组合性：不引入页面局部模式。
- 功能性：控件行为与底层 option 或状态一致。
- 可访问性：可聚焦、可取消、可读。
- 浏览器或运行时：UI 可实际渲染时必须验证。

## Gate 5 - 视觉回归

- UI-bearing work 需要 current 与 accepted 截图。
- 无 UI 变更的核心网络 feature 在 PRD 中写截图验收 N/A，并说明原因。
