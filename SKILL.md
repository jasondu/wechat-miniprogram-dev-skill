---
name: wechat-miniprogram-dev
description: 面向 CloudBase（云开发）的微信小程序开发技能。仅在任务明确涉及 wx.cloud、云函数、云数据库/云存储、cloudbaserc.json、CloudBase CLI 或 CloudBase 相关 CI/CD 时使用。非 CloudBase 的通用小程序开发优先使用 miniprogram-development。
---

# 微信小程序 CloudBase 开发技能

当用户的小程序项目明确使用云开发（如 `wx.cloud`、`cloudfunctions/`、`cloudbaserc.json`）时使用本技能。

## 适用范围

- 小程序端 `wx.cloud` 初始化与调用
- 云函数、云数据库、云存储相关开发
- CloudBase CLI 部署与环境注入
- 基于 CloudBase 的管理后台集成

## 不适用范围

- 不依赖 CloudBase 的纯页面/UI/路由开发
- 通用预览调试问题（优先 `miniprogram-development`）
- 与小程序 CloudBase 无关的纯后端任务

## Project Structure（推荐）

```text
project/
├── miniprogram/                 # 小程序代码
│   ├── pages/
│   ├── app.js
│   ├── app.json
│   └── app.wxss
├── cloudfunctions/              # 云函数
├── cloudbaserc.json             # 云函数/环境配置
├── project.config.json          # 小程序工程配置
└── .github/workflows/           # CI/CD（可选）
```

## 决策流程（必须遵循）

1. 先确认是否使用 CloudBase（看用户描述或仓库证据：`wx.cloud`、`cloudfunctions`、`cloudbaserc.json`）。
2. 若未使用 CloudBase，切换到通用小程序流程，不套用本技能规则。
3. 按场景按需加载一个参考文档：
   - 核心集成：`references/cloudbase-core.md`
   - CI/CD 与部署：`references/cloudbase-cicd.md`
   - 常见故障：`references/common-issues.md`
4. 只应用当前场景所需规则，避免一次性混用全部模式。

## 安全与约束

- `"auth": false` 仅用于明确需要公开访问、且低风险的接口。
- 公开云函数必须做参数校验、限流与最小权限控制。
- 不在代码中硬编码 `envId`、`appid`、密钥。
- runtime 与依赖版本按当前仓库实际约束校验，不盲目套示例版本。

## 执行前检查

- `project.config.json` 与小程序根目录配置一致。
- 环境 ID 注入策略明确（占位符替换或环境变量）。
- 页面配置文件与本地资源路径完整可用。
- 发布任务所需 `appid` / secrets 已通过安全配置提供。

## 参考文档

- [CloudBase 核心集成](references/cloudbase-core.md)
- [CloudBase CI/CD 与部署](references/cloudbase-cicd.md)
- [常见问题与修复](references/common-issues.md)

## 相关技能

- `miniprogram-development`：通用小程序开发/调试/预览
- `cloud-functions`：云函数运行时与实现细节
- `auth-wechat-miniprogram`：小程序认证流程
- `cloudbase-document-database-in-wechat-miniprogram`：数据库 SDK 用法
- `ai-model-wechat`：小程序 AI 能力集成
