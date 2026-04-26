# pages.json 规范摘要

## 关键官方结论

### 1. pages.json 的作用 [P0-强制]
- pages.json 用来对 uni-app 进行全局配置
- 决定页面文件的路径、窗口样式、原生导航栏、底部原生 tabbar 等
- **所有页面均需在 pages.json 中注册，否则不会被打包进应用**

### 2. tabBar 配置结构 [P0-强制]
```json
{
  "tabBar": {
    "color": "#999",
    "selectedColor": "#5AC725",
    "backgroundColor": "#F8F8F8",
    "position": "bottom",
    "borderStyle": "black",
    "list": [
      {
        "pagePath": "pages/index/index",
        "iconPath": "static/tabbar/index.png",
        "selectedIconPath": "static/tabbar/index-active.png",
        "text": "首页"
      },
      {
        "pagePath": "pages/list/list",
        "iconPath": "static/tabbar/list.png",
        "selectedIconPath": "static/tabbar/list-active.png",
        "text": "列表"
      }
    ]
  }
}
```

### 3. tabBar.pagePath 约束 [P0-强制]
- pagePath 必须是 pages 数组中已注册的页面路径
- pagePath 不能带参数（?queryString）
- tabBar.list 至少需要 2 个 item 才能正常显示

---

## 官方文档链接

| 文档 | 链接 |
|------|------|
| pages.json 全局配置 | https://uniapp.dcloud.net.cn/collocation/pages.html |
| pages.json 注册约束 | https://doc.dcloud.net.cn/uni-app-x/collocation/pagesjson.html |

---

## 典型项目结构

### 模式 A：根级 pages.json
```
project-root/
├── pages.json
├── src/
│   ├── pages/
│   │   ├── index/index.vue
│   │   ├── list/list.vue
│   │   └── user/user.vue
│   ├── components/
│   ├── App.vue
│   └── main.ts
├── static/
│   └── tabbar/
└── package.json
```

### 模式 B：src 内 pages.json
```
project-root/
├── src/
│   ├── pages.json
│   ├── pages/
│   │   ├── common/       # 公共分包（登录、网页等）
│   │   │   ├── login/index.vue
│   │   │   └── webview/index.vue
│   │   └── tab/          # 主包 tab 页
│   │       ├── index/index.vue
│   │       ├── list/list.vue
│   │       └── user/user.vue
│   ├── components/
│   ├── store/
│   └── utils/
├── manifest.json
└── package.json
```

---

## 代码检测要点

### ✅ 合规示例
```json
// pages.json
{
  "pages": [
    { "path": "pages/index/index" },
    { "path": "pages/list/list" },
    { "path": "pages/user/user" }
  ],
  "tabBar": {
    "list": [
      { "pagePath": "pages/index/index", "text": "首页" },
      { "pagePath": "pages/list/list", "text": "列表" },
      { "pagePath": "pages/user/user", "text": "我的" }
    ]
  }
}
```

### ❌ 违规示例
```json
// 错误1：pagePath 未注册
{
  "tabBar": {
    "list": [
      { "pagePath": "pages/home/home", "text": "首页" }  // pages/home/home 未在 pages 中注册
    ]
  }
}

// 错误2：带参数（switchTab 不支持）
{
  "tabBar": {
    "list": [
      { "pagePath": "pages/index/index?id=1", "text": "首页" }  // 不支持 query 参数
    ]
  }
}
```