# 支付宝小程序 customize tabBar 规范摘要

## 关键官方结论

### 1. 开启条件 [P0-强制]
- `tabBar.customize: true` 开启自定义模式
- `tabBar.overlay: true` 开启 Native 渲染模式（需同时配置 customize 为 true 才有效）
- 如果只配置 `overlay: true` 而未配置 `customize: true`，overlay 配置**无效**

### 2. overlay 模式说明 [P1]
- overlay 模式下，tabBar 使用 Native 渲染
- `getTabBar()` 返回的实例上有 `overlay` 属性（boolean），标识当前是否处于 overlay 模式
- 如果条件不满足，会**自动降级**为普通自定义 tabBar

```json
{
  "tabBar": {
    "customize": true,
    "overlay": true,
    "color": "#999999",
    "selectedColor": "#5AC725",
    "backgroundColor": "#F8F8F8",
    "list": [
      { "pagePath": "pages/index/index", "text": "首页" },
      { "pagePath": "pages/list/list", "text": "列表" },
      { "pagePath": "pages/user/user", "text": "我的" }
    ]
  }
}
```

### 3. 失效的 API [P1-强制]
- `my.hideTabBar()` 在 customize 模式下**失效**
- `my.setTabBarItem()` 等样式相关接口在 customize 模式下**失效**

### 4. 状态同步方式 [P0-强制]
```js
// 支付宝端：在页面 onShow 中同步
onShow() {
  const tabBar = this.getTabBar()
  if (tabBar) {
    tabBar.setData({ selected: 0 })
    // 可通过 tabBar.overlay 判断当前是否为 overlay 模式
    console.log('overlay mode:', tabBar.overlay)
  }
}
```

### 5. 与微信的差异点

| 行为 | 微信 custom | 支付宝 customize |
|------|------------|----------------|
| 配置字段 | `custom: true` | `customize: true` |
| overlay 支持 | 无 | `overlay: true`（需配合 customize）|
| hideTabBar 失效 | wx.hideTabBar | my.hideTabBar |
| Skyline 异步 | ✅ 支持 | 支付宝无 Skyline 模式 |

---

## 官方文档链接

| 文档 | 链接 |
|------|------|
| 自定义 tabBar | https://opendocs.alipay.com/mini/03jry7 |
| hideTabBar API | https://opendocs.alipay.com/mini/api/at18z8 |
| GitHub 备份文档 | https://github.com/AlipayDocs/open-docs/blob/main/mini/framework/%E5%9F%BA%E7%A1%80%E8%83%BD%E5%8A%9B/%E8%87%AA%E5%AE%9A%E4%B9%89%20tabBar.md |

---

## 常见错误写法

```js
// ❌ 错误1：只配置 overlay 未配置 customize
// app.json - 无效配置
{
  "tabBar": {
    "overlay": true,  // ❌ 需要同时配置 customize: true
    "list": [...]
  }
}

// ✅ 正确1：同时配置 customize 和 overlay
{
  "tabBar": {
    "customize": true,
    "overlay": true,
    "list": [...]
  }
}

// ❌ 错误2：依赖 my.hideTabBar 控制显隐
onShow() {
  my.hideTabBar({ animation: false })  // ❌ 在 customize 模式下失效
}

// ✅ 正确2：使用页面状态控制显隐，不要依赖 hideTabBar API
```

---

## 模拟器 ≠ 真机

**重要提示**：支付宝自定义 tabBar 在模拟器上的表现不一定等价于真机，特别是 overlay 模式的行为差异。**所有疑似问题必须在真机上验证**。