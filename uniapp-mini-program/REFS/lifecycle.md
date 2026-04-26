# 页面生命周期规范摘要

## 关键官方结论

### 1. onShow/onHide 在 tab 切换时的行为 [P0]
- **在 tabbar 页面，不同 tab 页面互相切换时，会触发各自的 onShow 和 onHide**
- onShow：页面每次出现在屏幕上都触发，包括从下级页面点返回露出当前页面
- onHide：当页面隐藏时触发（切到其他 tab 页、进入子页面、息屏等）

### 2. onLoad 的行为 [P0]
- onLoad 在页面首次加载时触发一次
- **tab 切换不会重复触发 onLoad**
- 如果在 onLoad 中做数据初始化，切换 tab 回当前页不会重新执行

### 3. onShow vs onLoad 选择 [P1-建议]
- 需要每次进入页面时刷新数据 → 使用 onShow
- 只在首次进入时执行一次 → 使用 onLoad
- tab 切换时需要刷新列表 → 在 onShow 中执行

---

## 官方文档链接

| 文档 | 链接 |
|------|------|
| 页面生命周期 | https://uniapp.dcloud.net.cn/tutorial/page.html |

---

## 生命周期时序图

```
用户打开应用
    ↓
[index/index] onLoad     ← 只触发一次
    ↓
[index/index] onShow   ← 首次显示
    ↓
[index/index] onReady  ← 首次渲染完成
    ↓
用户点击"列表"tab
    ↓
[index/index] onHide   ← 首页隐藏
    ↓
[list/list] onLoad     ← 列表页首次加载（如果之前没加载过）
    ↓ 或
[list/list] onShow     ← 列表页显示（如果之前已加载过）
    ↓
用户点击"首页"tab
    ↓
[list/list] onHide     ← 列表页隐藏
    ↓
[index/index] onShow    ← 首页再次显示（不会触发 onLoad）
```

---

## 常见错误写法

```js
// ❌ 错误1：在 onLoad 中执行每次切换需要刷新的逻辑
export default {
  onLoad() {
    // 这个只在首次加载时执行，tab 切换回来不会重新执行
    this.fetchUnreadCount()  // tab 切换不回刷新未读数
  }
}

// ✅ 正确1：将每次显示需要的逻辑放在 onShow
export default {
  onLoad() {
    // 只执行一次性的初始化（如读取缓存、设置静态数据）
    this.userInfo = uni.getStorageSync('userInfo')
  },
  onShow() {
    // 每次显示都执行（刷新动态数据）
    this.fetchUnreadCount()
  }
}

// ❌ 错误2：在 onShow 中做全量数据请求且没有做任何节流
export default {
  onShow() {
    this.fetchAllData()      // 每次切换都请求
    this.fetchBanner()       // 每次切换都请求
    this.fetchCategories()   // 每次切换都请求
    this.fetchRecommend()     // 每次切换都请求
    // → 触发 4 次 setData，列表会抖动 4 次
  }
}

// ✅ 正确2：合并请求或使用节流
export default {
  data() { return { needsRefresh: true } },
  onShow() {
    if (this.needsRefresh) {
      Promise.all([
        this.fetchAllData(),
        this.fetchBanner(),
        this.fetchCategories(),
        this.fetchRecommend()
      ]).then(([allData, banner, categories, recommend]) => {
        this.allData = allData
        this.banner = banner
        this.categories = categories
        this.recommend = recommend
        // 合并为一次 setData
      })
      this.needsRefresh = false
    }
  },
  onHide() {
    this.needsRefresh = true  // 离开页面时标记需要刷新
  }
}
```