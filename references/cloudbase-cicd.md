# CloudBase CI/CD 与部署

## 目录

1. Secrets 约定
2. 流水线结构
3. 常用部署命令
4. 分支与版本注意事项

## 1. Secrets 约定

建议在 CI 中提供：

- `TCB_ENV_ID`
- `TCB_SECRET_ID`
- `TCB_SECRET_KEY`
- `WX_MINIPROGRAM_APPID`

禁止把这些值写入仓库源码。

## 2. 流水线结构

典型步骤：

1. 构建/校验小程序资源。
2. 部署云函数。
3. 上传小程序包。
4. 部署管理后台静态站点（如存在）。

占位符替换示例：

```bash
sed -i "s/__TCB_ENV_ID__/${TCB_ENV_ID}/g" miniprogram/app.js
```

触发分支需与仓库实际一致（`main` 或 `master`）。

## 3. 常用部署命令

```bash
# 登录
 tcb login --apiKeyId <secretId> --apiKey <secretKey>

# 部署单个/全部云函数
 tcb fn deploy <functionName> -e <envId> --force
 tcb fn deploy all -e <envId> --force

# 上传小程序
 tcb miniprogram upload --filePath miniprogram -e <envId> --appId <appId>

# 部署静态托管
 tcb hosting deploy <localPath> -e <envId>
```

## 4. 分支与版本注意事项

- 在 CI 中先校验 CloudBase CLI 版本兼容性。
- GitHub Actions 建议固定 major 版本。
- 复制 workflow 前先确认默认分支名。
