# CloudBase 核心集成

## 目录

1. 小程序端初始化
2. 云函数基线
3. 数据库与云存储基线
4. 管理后台基线
5. 发布前核查

## 1. 小程序端初始化

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

说明：

- `__TCB_ENV_ID__` 建议在部署阶段替换。
- 依赖用户态的调用应在 `authPromise` 完成后再执行。

## 2. 云函数基线

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

规则：

- runtime / 依赖版本必须以仓库当前约束为准。
- 若使用 `auth: false`，必须额外做校验与防滥用控制。

## 3. 数据库与云存储基线

```javascript
const db = wx.cloud.database()
await db.collection('tasks').add({ data: { title: 'demo' } })
```

```javascript
const res = await wx.cloud.uploadFile({
  cloudPath: `images/${Date.now()}_demo.jpg`,
  filePath: tempFilePath
})
```

建议：数据库存 `fileID`，展示时动态调用 `getTempFileURL` 刷新临时链接。

## 4. 管理后台基线

```javascript
import cloudbase from '@cloudbase/js-sdk'

const app = cloudbase.init({ env: import.meta.env.VITE_CLOUDBASE_ENV })
const auth = app.auth()
const db = app.database()

await auth.signInAnonymously()
```

先登录，再访问数据库。

## 5. 发布前核查

- `project.config.json` 与 `miniprogramRoot` 一致。
- 环境 ID / appid 通过变量或 secrets 注入。
- 云函数权限最小化。
- 公共接口已做防刷与数据泄露检查。
