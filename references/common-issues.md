# 常见问题与修复

## 目录

1. 分享场景认证失败
2. TabBar 跳转报错
3. URL 参数过长
4. 图片链接过期

## 1. 分享场景认证失败（-501023）

问题：分享打开页面时用户态未就绪，调用受保护云函数失败。

修复：

```javascript
// app.js
this.authPromise = this.ensureAuth()

// page
async onLoad() {
  const app = getApp()
  await app.authPromise
}
```

若必须公开读接口，可在严格风控下设置 `auth: false`。

## 2. TabBar 跳转报错

问题：对 TabBar 页面使用了 `redirectTo`/`navigateTo`。

修复：

```javascript
wx.switchTab({ url: '/pages/index/index' })
```

## 3. URL 参数过长

问题：页面 query 承载数据过大。

修复：

```javascript
wx.setStorageSync('transfer_data', data)
wx.switchTab({ url: '/pages/target/target' })
```

目标页读取后及时清理，避免脏数据残留。

## 4. 图片链接过期

问题：临时链接失效导致分享页图片无法展示。

修复：

1. 数据库存 `fileID`。
2. 展示时通过 `getTempFileURL` 获取最新临时链接。
