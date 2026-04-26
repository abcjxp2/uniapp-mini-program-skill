# 微信小程序 custom tabBar 规范摘要

## 关键官方结论

### 1. 实例隔离机制 [P0-强制]
- **每个 tab 页下的自定义 tabBar 组件实例是相互隔离的**
- 不能跨页面共享同一个 tabBar 实例来控制状态
- 必须在当前页面通过 `getTabBar()` 获取实例

### 2. 选中态同步方式 [P0-强制]
> 如需实现 tab 选中态，要在当前页面下，通过 getTabBar 接口获取组件实例，并调用 setData 更新选中态。

```js
// 微信端：在页面 onShow 中同步
onShow() {
  if (typeof this.getTabBar === 'function' && this.getTabBar) {
    this.getTabBar((tabBar) => {
      tabBar.setData({ selected: 0 })  // 0=首页, 1=列表, 2=我的
    })
  }
}
```

### 3. app.json 配置要求 [P0-强制]
```json
{
  "tabBar": {
    "custom": true,
    "color": "#999999",
    "selectedColor": "#5AC725",
    "backgroundColor": "#F8F8F8",
    "borderStyle": "black",
    "list": [
      { "pagePath": "pages/index/index", "text": "首页" },
      { "pagePath": "pages/list/list", "text": "列表" },
      { "pagePath": "pages/user/user", "text": "我的" }
    ]
  }
}
```

### 4. 失效的 API [P1-强制]
- `wx.setTabBarItem` 在 custom 模式下**失效**
- `wx.hideTabBar` 等样式相关接口在 custom 模式下**失效**

### 5. Skyline 模式下的异步回调 [P1]
- Skyline 模式下 `getTabBar` 是**异步回调**方式
- 需要等待回调返回后才能调用 `setData`

```js
// Skyline 模式
onShow() {
  if (typeof this.getTabBar === 'function') {
    this.getTabBar((tabBar) => {
      tabBar.setData({ selected: 0 })
    })
  }
}
```

---

## 官方文档链接

| 文档 | 链接 |
|------|------|
| 自定义 tabBar（中文）| https://developers.weixin.qq.com/miniprogram/dev/framework/ability/custom-tabbar.html |
| 自定义 tabBar（英文）| https://developers.weixin.qq.com/miniprogram/en/dev/framework/ability/custom-tabbar.html |

---

## 常见错误写法

```js
// ❌ 错误1：跨页面共享 tabBar 实例
// home.vue - 在首页错误地创建了全局 tabBar 控制
const globalTabBar = getApp().globalData.tabBar
globalTabBar.setData({ selected: 0 })  // 无法跨页面生效，因为每页实例隔离

// ❌ 错误2：在 customTabBar 组件的 attached 中等待父组件传递 selectedIndex
// components/custom-tab-bar/index.js
Component({
  lifetimes: {
    attached() {
      // 这个 selected 是从页面 properties 传入的
      // 但时序上可能晚于组件首帧渲染，导致闪烁
      this.setData({ selected: this.properties.selected })
    }
  }
})

// ✅ 正确：在页面 onShow 中主动同步，不要依赖组件被动接收
// pages/index/index.vue
onShow() {
  if (this.getTabBar) {
    this.getTabBar((tabBar) => {
      tabBar.setData({ selected: 0 })
    })
  }
}
```

---

## 组件结构示例

```json
// custom-tab-bar/index.json
{
  "component": true,
  "usingComponents": {}
}

// custom-tab-bar/index.wxml
<view class="tab-bar">
  <view
    wx:for="{{list}}"
    wx:key="index"
    class="tab-item {{selected === index ? 'active' : ''}}"
    bindtap="onTabTap"
    data-index="{{index}}"
  >
    <image src="{{selected === index ? item.selectedIconPath : item.iconPath}}" />
    <text>{{item.text}}</text>
  </view>
</view>

// custom-tab-bar/index.js
Component({
  data: {
    selected: 0,
    list: []
  },
  methods: {
    onTabTap(e) {
      const { index } = e.currentTarget.dataset
      this.setData({ selected: index })
      // 触发页面跳转
      const pagePath = this.data.list[index].pagePath
      wx.switchTab({ url: pagePath })
    }
  }
})
```