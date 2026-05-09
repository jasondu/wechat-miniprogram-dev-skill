# Common Issues and Fixes

## TOC

1. Share-scene auth failure
2. TabBar navigation errors
3. Oversized URL params
4. Expired image URLs

## 1. Share-scene auth failure (-501023)

Problem: share entry opens page before user context is ready, protected function call fails.

Fix pattern:

```javascript
// app.js
this.authPromise = this.ensureAuth()

// page
async onLoad() {
  const app = getApp()
  await app.authPromise
}
```

For intentionally public read-only endpoints, use controlled `auth: false` plus validation.

## 2. TabBar navigation errors

Problem: `redirectTo` or `navigateTo` used for TabBar page.

Fix:

```javascript
wx.switchTab({ url: '/pages/index/index' })
```

## 3. Oversized URL params

Problem: payload too large for route query.

Fix:

```javascript
wx.setStorageSync('transfer_data', data)
wx.switchTab({ url: '/pages/target/target' })
```

Read and clear on target page to avoid stale data.

## 4. Expired image URLs

Problem: temporary file URL expires and shared pages fail to load image.

Fix:

1. Persist `fileID` in DB.
2. Resolve a fresh temporary URL with `getTempFileURL` at render time.
