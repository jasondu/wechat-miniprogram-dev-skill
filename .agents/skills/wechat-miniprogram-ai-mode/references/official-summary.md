# 微信小程序 AI 开发模式官方摘要

来源仅包含微信开放文档与官方 demo。

## 核心定位

- 这是“小程序 AI 开发模式（beta）”，不是 `wx.cloud.extend.AI`。
- 开发者把小程序能力抽象为原子接口、原子组件，并封装成小程序 AI `SKILL`。
- 小程序 AI 后台基于小程序 MCP 选择并调用这些能力。

## 官方文档确认的关键约束

- 当前处于内测阶段，相关代码不要合入正式版本送审。
- `AGENTS.md` 是全局提示词入口，`app.json > agent.instruction` 可配置。
- 一个小程序最多声明 `30` 个 skills。
- `SKILL.md` 最大 `16000` 字节。
- `mcp.json` 最大 `24000` 字节。
- `instruction` 文件最大 `10000` 字节。
- skill 必须放在独立分包里，并启用 `lazyCodeLoading: "requiredComponents"`。

## skill 结构

```text
skill/
|- SKILL.md
|- mcp.json
|- index.js
|- apis/
|- components/
```

## 运行时差异

- 原子接口、原子组件、实时动态组件运行在三个不同的 JS 上下文。
- 全局变量不共享。
- 半屏页更接近普通小程序页，但仍有能力限制。
- 组件渲染引擎不是普通小程序 WebView/Skyline 组合，需要注意样式差异。

## 新 API

原子接口侧：

- `wx.modelContext.createSkill`
- `skill.registerAPI`
- `skill.use`
- `wx.modelContext.getSessionId`
- `wx.modelContext.expireAllCards`

原子组件侧：

- `setRelatedPage`
- `sendFollowUpMessage`
- `openDetailPage`
- `preloadDetailPage`
- `expirePreviousCards`
- `on(NotificationType, callback)`

## 调试前提

- 微信开发者工具需要 Nightly Electron Build 最新版。
- 编译模式切到“小程序 AI 编译”。
- 调试基础库切到 `3.16.1`。
- 真机预览目前仅 iOS 支持，微信需 `8.0.74+`。

## FAQ 中最容易踩坑的点

- 安卓和鸿蒙暂不支持。
- AI 回复支持 markdown。
- AI 回复不支持流式输出。
- 文本和卡片不支持任意混排。
- 原子组件支持横滑，不支持竖滑。
- 不能靠全局状态给组件传数据。
- `uni-app` 可能导致 `wx.modelContext` 丢失。

## 最佳实践高优先级结论

- `mcp.json.description` 和 `inputSchema.description` 比 `SKILL.md` 更接近模型决策点。
- `SKILL.md` 主要写流程编排、跨接口规则、意图分流。
- 严格约束不要只写在 prompt 里，要在代码和 schema 中强校验。
- 不要编造 ID、枚举值、订单号等关键参数。

## 官方 demo 线索

官方 demo 当前仓库：

- `app.json`
- `skills/drink-skill/SKILL.md`
- `skills/drink-skill/mcp.json`
- `skills/drink-skill/index.js`

示例显示：

- `agent.skills[].path` 形如 `skills/drink-skill`
- `index.js` 通过 `wx.modelContext.createSkill('skills/drink-skill')` 创建 skill 并注册所有 API
- `SKILL.md` 更强调流程和铁律
- `mcp.json` 更强调单个 API 的触发边界与参数约束
