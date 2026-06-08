---
name: wechat-miniprogram-ai-mode
description: Use this skill when building, reviewing, or debugging WeChat Mini Program AI Development Mode (beta) integrations based on the official 微信开放文档. Covers AGENTS.md global instruction, app.json agent.skills registration, SKILL.md/mcp.json/index.js structure, wx.modelContext APIs, atomic APIs/components, runtime limits, debugging with WeChat DevTools Nightly, and best practices for writing Mini Program AI SKILLs.
alwaysApply: false
---

## When to use this skill

Use this skill when the user is working on **微信小程序 AI 开发模式（beta）** rather than ordinary Mini Program pages or `wx.cloud.extend.AI`.

Typical triggers:

- "接入微信小程序 AI 开发模式"
- "写 SKILL.md / mcp.json"
- "wx.modelContext 是什么"
- "原子接口 / 原子组件怎么写"
- "小程序 AI 调试不了怎么办"

Do **not** use this skill for:

- Plain Mini Program page/component development with no AI mode
- `wx.cloud.extend.AI` text generation in Mini Program
- Node.js or Web CloudBase AI SDK work

If the task is about general Mini Program engineering, pair this skill with `miniprogram-development`.

## Core distinction

This official doc describes **Mini Program AI Development Mode**, where the developer packages business capabilities into a Mini Program AI `SKILL`:

- `AGENTS.md`: global instruction for the whole Mini Program
- `SKILL.md`: detailed business rules for one skill
- `mcp.json`: callable atomic API declarations
- `index.js`: atomic API registration entry
- `apis/`: atomic API implementations
- `components/`: atomic component implementations

This is **not** the same as calling an LLM through `wx.cloud.extend.AI`.

## First-pass workflow

When asked to help on this topic, follow this order:

1. Confirm the project is targeting **小程序 AI 开发模式（beta）**.
2. Check whether the app has applied for AI Development Mode in 微信公众平台 or 微信开发者助手.
3. Inspect `app.json` for `lazyCodeLoading`, independent subpackages, and `agent.skills`.
4. Inspect `AGENTS.md`, each `SKILL.md`, `mcp.json`, `index.js`, `apis/`, and `components/`.
5. Validate that every declared API in `mcp.json` is registered in `index.js` and implemented in `apis/`.
6. Validate that any UI-bound API uses `_meta.ui.componentPath` and that the component path exists.
7. Check runtime restrictions before suggesting APIs or component behaviors.
8. When debugging, target **微信开发者工具 Nightly** and the AI compile mode first.

## Required app-level setup

`app.json` should include:

- `lazyCodeLoading: "requiredComponents"`
- `subPackages` with an **independent** package for AI skills
- `agent.skills` declarations
- Optional `agent.instruction` pointing to `AGENTS.md`
- Optional `agent.pageMetadata` for text-link fallback

Example shape:

```json
{
  "lazyCodeLoading": "requiredComponents",
  "subPackages": [
    {
      "root": "skills",
      "pages": [],
      "independent": true
    }
  ],
  "agent": {
    "instruction": "AGENTS.md",
    "skills": [
      {
        "name": "drink",
        "description": "点单场景",
        "path": "skills/drink-skill"
      }
    ]
  }
}
```

Current documented limits:

- `agent.instruction` file size: max `10000` bytes
- One Mini Program can declare at most `30` skills

## Required skill package structure

Each Mini Program AI skill should live inside one independent subpackage and include:

```text
path/to/skill/
|- SKILL.md
|- mcp.json
|- index.js
|- apis/
|- components/
```

Constraints from the official docs:

- `SKILL.md` is a **single file only**; do not assume the runtime supports importing other markdown files
- `SKILL.md` max size: `16000` bytes
- `mcp.json` max size: `24000` bytes
- `mcp.json` size calculation ignores `outputSchema` plus whitespace/newlines

## What goes where

Use the documented attention hierarchy:

1. Atomic API return `content`: strongest for immediate next-step guidance
2. `mcp.json` `description`: strongest for API selection boundaries
3. `mcp.json` `inputSchema.properties.*.description`: strongest for argument generation
4. `SKILL.md`: best for orchestration, routing, cross-API rules

Write content accordingly:

- Put API purpose, trigger conditions, and forbidden cases in `mcp.json.description`
- Put parameter meaning and hard constraints in `inputSchema`
- Put workflow, sequencing, intent routing, and shared rules in `SKILL.md`
- Put actual result plus next-step hint in returned `content`

Do **not** bloat `SKILL.md` with per-API detail that belongs in `mcp.json`.

## `mcp.json` authoring rules

Each atomic API entry should include:

- `name`: must match the registered API name in `index.js`
- `description`: clear business object, trigger condition, and forbidden scenarios
- `inputSchema`: JSON Schema object
- `outputSchema`: recommended
- Optional `_meta.ui.componentPath` for component rendering

Strong guidance:

- Use semantic API names such as `searchFlights`, not vague names like `search`
- First sentence should name the **business object**, not the parameter list
- Avoid implementation details in descriptions
- Keep API responsibilities non-overlapping
- Put prohibitions early, not buried in the middle
- Use JSON Schema enums/min/max/required aggressively to constrain model arguments

## `index.js` registration rules

Register APIs through `wx.modelContext.createSkill(skillPath)`:

```js
const getWeather = require('./apis/getWeather')

const skill = wx.modelContext.createSkill('skills/weather-skill')
skill.registerAPI('getWeather', getWeather)
```

Important:

- `skillPath` must match the declared skill path
- Registered API names must match `mcp.json`
- Use `skill.use(...)` middleware for shared login, logging, error handling, or auth reuse

Middleware context includes:

- `name`
- `skillPath`
- `arguments`

## Atomic API implementation expectations

Atomic APIs are the smallest executable business unit. Treat them as:

- Single-purpose
- Strongly validated
- Structured in input and output
- Safe against hallucinated IDs or parameters

Recommend these patterns:

- Validate all externally supplied IDs against upstream results or storage
- Reuse login/session checks via middleware
- Return structured data for components through `structuredContent`
- Return explicit next-step hints in `content`
- Enforce business invariants in code, not just in prompt text

## Atomic component rules

Atomic components render GUI cards in the conversation flow.

Supported built-in components currently include:

- `view`
- `text` without `user-select`
- `map` with limited interaction
- `button` without `open-type`
- `image` using network `png`/`jpg`
- `canvas` with `2d` only
- `scroll-view` with horizontal scroll only

Key constraints:

- Width is fixed by screen width
- Height range is constrained from `4:1` to `1:1`
- Height is determined at initialization and cannot change later
- Only limited events are supported; mainly tap plus image load/error
- No vertical scrolling
- No animation
- Components cannot open the Mini Program directly

Use `components[].relatedPage` in `mcp.json` and `setRelatedPage({ path, query })` when the card needs a corresponding Mini Program page.

## Real-time dynamic components

Default atomic components do **not** support:

- `wx.request`
- Cloud APIs
- timers such as `setTimeout` / `setInterval`

If the scene genuinely requires live updates, declare the component as dynamic in `mcp.json` and note that this capability needs separate review. Use it sparingly.

## Runtime model and context isolation

The execution environment is separate from the normal Mini Program runtime.

Critical implications:

- Atomic APIs, atomic components, and dynamic components run in different JavaScript contexts
- Global variables are **not shared** across those contexts
- Data must be passed through API return fields such as `content`, `structuredContent`, and `_meta`
- Half-screen pages are closer to normal Mini Program pages, but some capabilities remain restricted

Never suggest using shared global state between atomic APIs and components.

## New `wx.modelContext` APIs

Atomic API side:

- `wx.modelContext.createSkill(skillPath)`
- `skill.registerAPI(name, handler)`
- `skill.use(middleware)`
- `wx.modelContext.getSessionId()`
- `wx.modelContext.expireAllCards({ componentPaths, match })`

Atomic component side:

- `wx.modelContext.getViewContext(this).setRelatedPage({ path, query })`
- `wx.modelContext.getContext().sendFollowUpMessage()`
- `wx.modelContext.getViewContext(this).openDetailPage({ url })`
- `wx.modelContext.getViewContext(this).preloadDetailPage({ url })`
- `wx.modelContext.getViewContext(this).expirePreviousCards({ componentPaths, match })`
- `wx.modelContext.getViewContext(this).on(NotificationType, callback)`

Useful notifications:

- `Input`
- `Result`
- `Overflow`
- `Expire`

## Existing API support guidance

Do not assume normal Mini Program APIs all work everywhere.

High-value rules from the docs:

- Atomic APIs support many business APIs, including `wx.login`, `wx.request`, `wx.cloud.*`, storage APIs, payment APIs, and more
- Atomic components support a much narrower set
- Storage APIs are supported in both contexts and are the documented way to reuse login credentials
- Some APIs become available in components only when declared dynamic

When unsure, verify against the official API support table before proposing an implementation.

## Login and identity

The user identity is shared with the original Mini Program.

Preferred options:

- Reuse login credentials through storage APIs
- Complete login flows through supported APIs such as `wx.login` and `wx.getPhoneNumber`
- Centralize repeated auth logic in middleware

## Debugging requirements

Use the official debugging path first:

- **微信开发者工具 Nightly Electron Build 最新版本**
- Compile mode: **小程序 AI 编译**
- Base library: `3.16.1`

If the entry is missing:

- Re-login or reopen the project window
- Switch base library away from `3.16.1` and back to force resource download

Current preview support from the docs:

- Real-device preview requires WeChat `8.0.74+`
- Currently **iOS only**
- Base library on device should be `3.16.1+`
- Remote "真机调试" is not supported for this mode

## Best-practice rules to enforce

- Let AI handle fuzzy intent, but keep execution through explicit atomic APIs
- Do not skip required intermediate APIs just because the model can guess the next step
- Keep card data inside cards; avoid re-expanding card details as markdown lists in plain text
- Never invent IDs, enum values, or order numbers
- If a constraint is hard, enforce it in code and `inputSchema`, not only in `SKILL.md`
- Realtime service progress should update cards rather than relying on static cards
- Use text-links only as fallback; core flows should stay inside AI mode

## Common pitfalls

- Confusing this mode with `wx.cloud.extend.AI`
- Forgetting `lazyCodeLoading: "requiredComponents"`
- Declaring a skill outside an independent subpackage
- `mcp.json` API names not matching `index.js`
- Omitting related page config for components
- Passing data through globals across contexts
- Using unsupported component APIs or vertical scrolling
- Assuming streaming text replies are supported; the FAQ says they are **not**
- Assuming arbitrary text/card interleaving is supported; current format is limited
- Missing `wx.modelContext` because `uni-app` replaced global `wx`

If `wx.modelContext` is `undefined` in a `uni-app` based project, inspect `@dcloudio/uni-mp-weixin/dist/wx.js` and ensure `modelContext` is preserved.

## Review checklist

When reviewing a Mini Program AI skill, check:

- `app.json` has the right `agent` and subpackage setup
- `AGENTS.md` scopes the product and routes skills clearly
- `SKILL.md` focuses on orchestration instead of duplicating `mcp.json`
- `mcp.json` descriptions and `inputSchema` are concrete and restrictive
- All IDs and enums come from real upstream data
- Every declared component path exists
- Related pages and half-screen flows are wired correctly
- Cross-context data passing is explicit
- DevTools/preview prerequisites are documented for the team

## Official references

Prefer official WeChat sources:

- Guide: `https://developers.weixin.qq.com/miniprogram/dev/ai/guide.html`
- Integration: `https://developers.weixin.qq.com/miniprogram/dev/ai/integration.html`
- Runtime: `https://developers.weixin.qq.com/miniprogram/dev/ai/operating-mechanism.html`
- Debugging: `https://developers.weixin.qq.com/miniprogram/dev/ai/debugging.html`
- Best practices: `https://developers.weixin.qq.com/miniprogram/dev/ai/best-practices.html`
- API support: `https://developers.weixin.qq.com/miniprogram/dev/ai/reference/api.html`
- FAQ: `https://developers.weixin.qq.com/miniprogram/dev/ai/faq.html`
- Official demo: `https://github.com/wechat-miniprogram/ai-mode-demo`
