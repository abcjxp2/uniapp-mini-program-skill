# switchTab API 规范摘要

## 关键官方结论

### 1. switchTab 约束 [P0-强制]
- **目标必须是 tabBar 页面**（在 pages.json 的 tabBar.list 中定义）
- **url 不能带参数**（路径后不能带 query 字符串）
- 跳转 tabBar 页面只能使用 switchTab，不能使用 navigateTo/redirectTo

### 2. 官方描述
> url: 需要跳转的 tabBar 页面的路径（需在 pages.json 的 tabBar 字段定义的页面），路径后不能带参数

### 3. onLoad 不会在 tab 切换时触发
- tab 页之间互相切换时，触发各自的 onShow 和 onHide
- onLoad 只在页面首次加载时触发，tab 切换不会重复触发 onLoad
- 如果需要在每次显示时刷新数据，应使用 onShow

---

## 官方文档链接

| 文档 | 链接 |
|------|------|
| switchTab API | https://uniapp.dcloud.net.cn/api/router.html#switchtab |
| 页面生命周期 | https://uniapp.dcloud.net.cn/tutorial/page.html |

---

## 合规用法

```js
// ✅ 正确：无参数
uni.switchTab({ url: '/pages/index/index' })
uni.switchTab({ url: '/pages/list/list' })
uni.switchTab({ url: '/pages/user/user' })
```

## 违规用法

```js
// ❌ 错误1：带参数（官方明确不支持）
uni.switchTab({ url: '/pages/index/index?id=1' })
uni.switchTab({ url: '/pages/index/index?from=splash' })

// ❌ 错误2：使用 navigateTo 进入 tab 页
uni.navigateTo({ url: '/pages/index/index' })

// ❌ 错误3：使用 redirectTo 进入 tab 页
uni.redirectTo({ url: '/pages/index/index' })
```

---

## tab 切换生命周期行为

| 生命周期 | 触发时机 | tab 切换时是否触发 |
|----------|----------|-----------------|
| onLoad | 页面首次加载 | ❌ 否 |
| onShow | 页面显示 | ✅ 是（每次切回都触发）|
| onHide | 页面隐藏 | ✅ 是 |
| onReady | 页面初次渲染完成 | ❌ 否（首次加载时触发一次）|

---

## 数据传递方案

switchTab 不支持 url 参数，因此 tab 页之间的数据共享方案：

1. **全局状态管理**：Pinia/Vuex
2. **本地存储**：uni.setStorageSync / uni.getStorageSync
3. **页面通信**：eventChannel（但需在首次建立连接时）
4. **全局变量**：App.vue 中定义的响应式数据

```js
// 推荐：全局状态管理
// store/tab.js
import { defineStore } from 'pinia'
export const useTabStore = defineStore('tab', {
  state: () => ({ currentIndex: 0, data: {} }),
  actions: {
    setData(key, value) { this.data[key] = value }
  }
})

// 页面 A
const tabStore = useTabStore()
tabStore.setData('userInfo', { name: 'John' })

// 页面 B
const tabStore = useTabStore()
console.log(tabStore.data.userInfo) // { name: 'John' }
```