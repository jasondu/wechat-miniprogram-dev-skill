---
name: wechat-miniprogram-dev
description: WeChat Mini Program (微信小程序) development with CloudBase (云开发), including cloud functions, database, storage, CI/CD deployment, and admin dashboard. Use this skill when developing WeChat Mini Programs, working with wx.cloud APIs, deploying cloud functions, setting up GitHub Actions CI/CD, or building admin dashboards for mini program management. TRIGGER on: "小程序", "云开发", "wx.cloud", "云函数", "cloudbase", "微信小程序", "CloudBase", "miniprogram", "tabbar", "分享场景".
---

# WeChat Mini Program Development with CloudBase

A comprehensive guide for developing WeChat Mini Programs with Tencent CloudBase (云开发), covering project structure, cloud functions, database operations, storage handling, CI/CD deployment, and admin dashboard development.

## Project Structure

```
project/
├── miniprogram/                 # 小程序代码
│   ├── pages/                   # 页面
│   │   ├── index/
│   │   │   ├── index.js
│   │   │   ├── index.wxml
│   │   │   ├── index.wxss
│   │   │   └── index.json
│   │   └── ...
│   ├── app.js                   # 小程序入口
│   ├── app.json                 # 配置文件
│   └── app.wxss                 # 全局样式
├── cloudfunctions/              # 云函数
│   ├── getUserInfo/
│   │   ├── index.js
│   │   └── package.json
│   └── ...
├── admin/                       # 管理后台 (可选)
│   ├── src/
│   │   ├── api/
│   │   ├── views/
│   │   └── components/
│   └── package.json
├── cloudbaserc.json             # 云函数配置
├── project.config.json          # 小程序配置
└── .github/workflows/           # CI/CD
```

## CloudBase Initialization

### 小程序端初始化

在 `app.js` 中初始化云开发：

```javascript
App({
  onLaunch() {
    wx.cloud.init({
      env: '__TCB_ENV_ID__',  // 构建时替换为实际环境ID
      traceUser: true
    })

    // 分享场景认证 Promise
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

**重要**: 使用 `__TCB_ENV_ID__` 占位符，CI/CD 时替换为实际环境 ID。

### 管理后台初始化

使用 CloudBase JS SDK，需要先匿名登录：

```javascript
import cloudbase from '@cloudbase/js-sdk'

const ENV_ID = import.meta.env.VITE_CLOUDBASE_ENV

const app = cloudbase.init({ env: ENV_ID })
const auth = app.auth()
const db = app.database()

// 必须先匿名登录才能访问数据库
async function anonymousLogin() {
  await auth.signInAnonymously()
}
```

**在 App.vue 初始化时调用匿名登录**：

```javascript
onMounted(async () => {
  await anonymousLogin()
})
```

## Cloud Functions (云函数)

### 云函数结构

```javascript
// cloudfunctions/getUserInfo/index.js
const cloud = require('wx-server-sdk')

cloud.init({ env: cloud.DYNAMIC_CURRENT_ENV })

const db = cloud.database()

exports.main = async (event, context) => {
  const { OPENID } = cloud.getWXContext()

  try {
    // 业务逻辑
    const result = await db.collection('users').where({ _openid: OPENID }).get()

    return {
      success: true,
      data: result.data
    }
  } catch (error) {
    console.error('Error:', error)
    return {
      success: false,
      error: error.message
    }
  }
}
```

### package.json

```json
{
  "name": "getUserInfo",
  "version": "1.0.0",
  "main": "index.js",
  "dependencies": {
    "wx-server-sdk": "~2.6.3"
  }
}
```

### cloudbaserc.json 配置

```json
{
  "version": "2.0",
  "envId": "{{envId}}",
  "functionRoot": "./cloudfunctions",
  "functions": [
    {
      "name": "getUserInfo",
      "handler": "index.main",
      "runtime": "Nodejs18.15",
      "timeout": 15,
      "memorySize": 128,
      "installDependency": true
    }
  ]
}
```

### 公开访问配置

分享场景需要公开访问的云函数，添加 `"auth": false`：

```json
{
  "name": "queryImageTask",
  "auth": false  // 无需登录即可调用
}
```

## Database Operations

### 小程序端数据库操作

```javascript
const db = wx.cloud.database()

// 查询
db.collection('users').where({ _openid: '{openid}' }).get()

// 添加
db.collection('tasks').add({ data: { ... } })

// 更新
db.collection('tasks').doc(id).update({ data: { ... } })

// 分页查询
db.collection('tasks')
  .orderBy('createTime', 'desc')
  .skip((page - 1) * pageSize)
  .limit(pageSize)
  .get()
```

### 管理后台数据库操作

```javascript
// 需要先匿名登录
const db = app.database()

// 查询
db.collection('users').get()

// 条件查询
db.collection('tasks').where({ status: 'processing' }).get()
```

## Cloud Storage (云存储)

### 图片上传

```javascript
wx.cloud.uploadFile({
  cloudPath: `images/${Date.now()}_${Math.random().toString(36).substr(2, 9)}.jpg`,
  filePath: tempFilePath  // 本地临时路径
}).then(res => {
  console.log('fileID:', res.fileID)
})
```

### 获取临时 URL (重要!)

**tempFileURL 会在 2 小时后过期**，需要用 fileID 刷新：

```javascript
// 从 fileID 获取最新临时 URL
const tempUrlRes = await wx.cloud.getTempFileURL({
  fileList: [fileID]
})
const imageUrl = tempUrlRes.fileList[0].tempFileURL
```

**最佳实践**: 数据库存储 fileID，每次展示时获取最新 tempFileURL。

## Common Issues & Solutions

### 1. 分享场景权限问题 (-501023)

分享页面打开时用户未认证，调用云函数会失败。

**解决方案**:

```javascript
// app.js
this.authPromise = this.ensureAuth()

// 页面中等待认证完成
async onLoad() {
  const app = getApp()
  await app.authPromise  // 等待认证
  // 然后再调用云函数
}
```

配合云函数配置 `"auth": false`。

### 2. TabBar 页面跳转失败

错误: `redirectTo:fail can not redirect to a tabbar page`

**解决方案**: TabBar 页面使用 `switchTab`：

```javascript
// ❌ 错误
wx.redirectTo({ url: '/pages/index/index' })

// ✅ 正确
wx.switchTab({ url: '/pages/index/index' })
```

### 3. URL 参数过长

小程序页面参数通过 URL 传递，长度有限制。

**解决方案**: 使用 storage 传递：

```javascript
// 存储数据
wx.setStorageSync('transfer_data', data)

// 跳转
wx.switchTab({ url: '/pages/target/target' })

// 目标页面读取
onShow() {
  const data = wx.getStorageSync('transfer_data')
  if (data) {
    this.setData({ ...data })
    wx.removeStorageSync('transfer_data')  // 清除避免残留
  }
}
```

### 4. 图片链接过期

tempFileURL 2 小时后过期，分享场景无法加载图片。

**解决方案**:

```javascript
// 数据库存储 fileID
await db.collection('tasks').add({
  data: {
    imageFileID: fileID,  // 存储 fileID
    imageUrl: tempUrl     // 临时 URL（可选）
  }
})

// 展示时刷新
async loadImage() {
  const { result } = await wx.cloud.callFunction({
    name: 'queryTask',
    data: { taskId }
  })

  if (result.imageFileID) {
    const tempUrlRes = await wx.cloud.getTempFileURL({
      fileList: [result.imageFileID]
    })
    imageUrl = tempUrlRes.fileList[0].tempFileURL
  }
}
```

## CI/CD with GitHub Actions

### 必要的 Secrets

在 GitHub Repository Settings → Secrets and variables → Actions 中配置：

- `TCB_ENV_ID`: 云开发环境 ID
- `TCB_SECRET_ID`: 腾讯云 API SecretId
- `TCB_SECRET_KEY`: 腾讯云 API SecretKey
- `WX_MINIPROGRAM_APPID`: 小程序 APPID

### Deploy Workflow 示例

```yaml
name: Deploy

on:
  push:
    branches: [master]

jobs:
  deploy-miniprogram:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Replace placeholders
        run: |
          sed -i "s/__TCB_ENV_ID__/${{ secrets.TCB_ENV_ID }}/g" miniprogram/app.js

      - name: Upload miniprogram
        run: |
          tcb miniprogram upload --filePath miniprogram \
            -e ${{ secrets.TCB_ENV_ID }} \
            --appId ${{ secrets.WX_MINIPROGRAM_APPID }}
        env:
          TCB_SECRET_ID: ${{ secrets.TCB_SECRET_ID }}
          TCB_SECRET_KEY: ${{ secrets.TCB_SECRET_KEY }}

  deploy-cloudfunctions:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install CloudBase CLI
        run: npm install -g @cloudbase/cli

      - name: Deploy functions
        run: |
          tcb login --apiKeyId ${{ secrets.TCB_SECRET_ID }} --apiKey ${{ secrets.TCB_SECRET_KEY }}
          tcb fn deploy all -e ${{ secrets.TCB_ENV_ID }} --force

  deploy-admin:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build admin
        working-directory: admin
        run: npm run build
        env:
          VITE_CLOUDBASE_ENV: ${{ secrets.TCB_ENV_ID }}
          VITE_MINIPROGRAM_APPID: ${{ secrets.WX_MINIPROGRAM_APPID }}

      - name: Deploy to static hosting
        run: |
          tcb hosting deploy admin/dist -e ${{ secrets.TCB_ENV_ID }}
```

### 管理后台环境变量

管理后台需要配置 `.env` 或环境变量：

```
VITE_CLOUDBASE_ENV=cloudbase-xxxxxx
VITE_MINIPROGRAM_APPID=wx1234567890abcdef
```

本地开发创建 `.env.local` 文件。

## Admin Dashboard Development

### Vue 3 + CloudBase JS SDK

```javascript
// src/api/cloudbase.js
import cloudbase from '@cloudbase/js-sdk'

const ENV_ID = import.meta.env.VITE_CLOUDBASE_ENV
const MINIPROGRAM_APPID = import.meta.env.VITE_MINIPROGRAM_APPID

const app = cloudbase.init({ env: ENV_ID })
const auth = app.auth()
const db = app.database()

// 匿名登录（必须）
export async function anonymousLogin() {
  await auth.signInAnonymously()
}

// 数据库操作示例
export async function getUsers(page = 1, pageSize = 20) {
  const skip = (page - 1) * pageSize
  const res = await db.collection('users')
    .orderBy('createTime', 'desc')
    .skip(skip)
    .limit(pageSize)
    .get()
  return { list: res.data, total: res.data.length }
}
```

### App.vue 初始化

```vue
<script setup>
import { onMounted } from 'vue'
import { anonymousLogin } from './api/cloudbase'

onMounted(async () => {
  await anonymousLogin()
})
</script>
```

## Deployment Commands

### CloudBase CLI 常用命令

```bash
# 登录
tcb login --apiKeyId <secretId> --apiKey <secretKey>

# 部署云函数
tcb fn deploy <functionName> -e <envId> --force

# 部署所有云函数
tcb fn deploy all -e <envId> --force

# 部署静态网站
tcb hosting deploy <localPath> -e <envId>

# 上传小程序
tcb miniprogram upload --filePath miniprogram -e <envId> --appId <appId>

# 查看云函数列表
tcb fn list -e <envId>
```

## Best Practices Summary

1. **环境 ID**: 使用占位符 `__TCB_ENV_ID__`，CI/CD 替换
2. **分享认证**: app.js 创建 authPromise，页面等待
3. **图片存储**: 存 fileID，展示时 getTempFileURL
4. **TabBar 跳转**: 使用 switchTab，不能用 redirectTo/navigateTo
5. **大数据传递**: 使用 storage，不用 URL 参数
6. **管理后台**: 先匿名登录，再访问数据库
7. **环境变量**: 管理后台用 VITE_CLOUDBASE_ENV 等
8. **公开云函数**: 配置 `"auth": false`

## Related Skills

- `ai-model-wechat`: AI capabilities in WeChat Mini Program
- `auth-wechat-miniprogram`: Authentication patterns
- `cloud-functions`: CloudBase cloud functions details
- `cloudbase-document-database-in-wechat-miniprogram`: Database SDK usage