# CloudBase CI/CD and Deployment

## TOC

1. Secrets contract
2. Pipeline structure
3. Deployment commands
4. Version and branch notes

## 1. Secrets contract

Recommended CI secrets:

- `TCB_ENV_ID`
- `TCB_SECRET_ID`
- `TCB_SECRET_KEY`
- `WX_MINIPROGRAM_APPID`

Never commit these values into repository files.

## 2. Pipeline structure

Typical jobs:

1. Build/verify mini program assets.
2. Deploy cloud functions.
3. Upload mini program package.
4. Deploy admin static site (if present).

Sample placeholder replacement:

```bash
sed -i "s/__TCB_ENV_ID__/${TCB_ENV_ID}/g" miniprogram/app.js
```

Use branch triggers that match the current repository convention (`main` or `master`).

## 3. Deployment commands

```bash
# Login
 tcb login --apiKeyId <secretId> --apiKey <secretKey>

# Deploy one/all functions
 tcb fn deploy <functionName> -e <envId> --force
 tcb fn deploy all -e <envId> --force

# Upload mini program
 tcb miniprogram upload --filePath miniprogram -e <envId> --appId <appId>

# Deploy static hosting
 tcb hosting deploy <localPath> -e <envId>
```

## 4. Version and branch notes

- Validate CloudBase CLI compatibility in CI before rollout.
- Pin major versions of CI actions where possible.
- Confirm default branch name before copying workflow examples.
