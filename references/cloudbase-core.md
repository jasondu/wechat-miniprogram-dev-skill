# CloudBase Core Integration

## TOC

1. Initialization
2. Cloud function baseline
3. Database and storage baseline
4. Admin dashboard baseline
5. Release-time checks

## 1. Initialization

Initialize CloudBase in mini program `app.js`:

```javascript
App({
  onLaunch() {
    wx.cloud.init({
      env: '__TCB_ENV_ID__',
      traceUser: true
    })

    this.authPromise = this.ensureAuth()
  },

  ensureAuth() {
    return new Promise((resolve) => {
      wx.cloud.callFunction({
        name: 'getUserInfo',
        success: () => resolve(true),
        fail: () => resolve(false)
      })
    })
  }
})
```

Notes:

- Keep `__TCB_ENV_ID__` as a deployment-time placeholder.
- Do not call protected functions before `authPromise` resolves.

## 2. Cloud function baseline

```javascript
const cloud = require('wx-server-sdk')
cloud.init({ env: cloud.DYNAMIC_CURRENT_ENV })

const db = cloud.database()

exports.main = async () => {
  const { OPENID } = cloud.getWXContext()
  const result = await db.collection('users').where({ _openid: OPENID }).get()
  return { success: true, data: result.data }
}
```

Function config guidance:

- Runtime and dependency versions must match current project constraints.
- If setting `auth: false`, add explicit request validation and abuse controls.

## 3. Database and storage baseline

Mini program DB:

```javascript
const db = wx.cloud.database()
await db.collection('tasks').add({ data: { title: 'demo' } })
```

Storage upload:

```javascript
const res = await wx.cloud.uploadFile({
  cloudPath: `images/${Date.now()}_demo.jpg`,
  filePath: tempFilePath
})
```

Use `fileID` as source of truth. Temporary URLs can expire; refresh with `getTempFileURL` when rendering.

## 4. Admin dashboard baseline

With `@cloudbase/js-sdk`, authenticate before DB access:

```javascript
import cloudbase from '@cloudbase/js-sdk'

const app = cloudbase.init({ env: import.meta.env.VITE_CLOUDBASE_ENV })
const auth = app.auth()
const db = app.database()

await auth.signInAnonymously()
```

## 5. Release-time checks

- `project.config.json` and `miniprogramRoot` are aligned.
- Environment IDs and app IDs come from env/secrets, not constants.
- Function permissions are least privilege.
- Public endpoints are audited for abuse and data leakage.
