---
name: wechat-miniprogram-dev
description: CloudBase-focused WeChat Mini Program development skill. Use when tasks explicitly involve CloudBase or wx.cloud in mini programs: cloud functions, cloud database/storage, cloudbaserc.json, CloudBase CLI, or CI/CD deployment for CloudBase resources. Prefer generic miniprogram-development for non-CloudBase mini program work.
---

# WeChat Mini Program Development with CloudBase

Use this skill when the user is working on a WeChat Mini Program that explicitly uses CloudBase (云开发), such as `wx.cloud`, cloud functions, CloudBase database/storage, or CloudBase deployment pipelines.

## When to use

Use this skill for:

- `wx.cloud` initialization, cloud functions, cloud database, cloud storage
- `cloudfunctions/`, `cloudbaserc.json`, CloudBase CLI workflows
- CloudBase-related CI/CD and environment injection
- CloudBase-backed admin dashboard integration

Do not use this skill for:

- Pure mini program UI/page logic without CloudBase dependency
- Generic mini program preview/debug questions better covered by `miniprogram-development`
- Pure backend tasks not tied to mini program CloudBase usage

## Decision flow (required)

1. Confirm CloudBase usage from user intent or repo evidence (`wx.cloud`, `cloudfunctions`, `cloudbaserc.json`).
2. If CloudBase is not present, switch to generic mini program workflow.
3. Pick one scenario and load only the needed reference:
   - Core integration: `references/cloudbase-core.md`
   - CI/CD and deployment: `references/cloudbase-cicd.md`
   - Known issues and fixes: `references/common-issues.md`
4. Apply only scenario-relevant guidance; avoid mixing all patterns by default.

## Guardrails

- Security first: `"auth": false` is only for intentionally public, low-risk endpoints.
- For any public cloud function, require input validation, rate limiting, and least-privilege data access.
- Do not hardcode environment IDs, app IDs, or credentials in source.
- Treat runtime versions and dependency versions as project-specific; verify against the current repository/toolchain before changing.

## Project checks

Before implementation or release advice, validate:

- `project.config.json` exists and points to the expected mini program root
- CloudBase environment selection strategy is clear (placeholder or env vars)
- Referenced local assets and page config files exist
- For release-related tasks, required appid/secrets are available via secure config

## References

- [CloudBase Core Integration](references/cloudbase-core.md)
- [CloudBase CI/CD and Deployment](references/cloudbase-cicd.md)
- [Common Issues and Fixes](references/common-issues.md)

## Related skills

- `miniprogram-development`: generic mini program development/debug/preview flow
- `cloud-functions`: detailed cloud function development/runtime concerns
- `auth-wechat-miniprogram`: mini program authentication patterns
- `cloudbase-document-database-in-wechat-miniprogram`: DB SDK query/update patterns
- `ai-model-wechat`: AI model integration in mini programs
