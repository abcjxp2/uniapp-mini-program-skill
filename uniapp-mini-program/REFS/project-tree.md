# uni-app 典型项目结构

## 模式 A：根级 pages.json（Vue CLI / Vite 模版）

```
project-root/
├── pages.json              # 页面路由 + tabBar 配置
├── src/
│   ├── pages/
│   │   ├── index/
│   │   │   └── index.vue    # 首页 tabBar 页
│   │   ├── list/
│   │   │   └── list.vue     # 列表 tabBar 页
│   │   ├── user/
│   │   │   └── user.vue     # 我的 tabBar 页
│   │   └── common/          # 公共页面（非 tabBar）
│   │       ├── login/
│   │       │   └── index.vue
│   │       └── webview/
│   │           └── index.vue
│   ├── static/
│   │   └── tabbar/          # tabBar 图标
│   │       ├── index.png
│   │       ├── index-active.png
│   │       ├── list.png
│   │       ├── list-active.png
│   │       ├── user.png
│   │       └── user-active.png
│   ├── components/          # 自定义组件
│   ├── App.vue
│   ├── main.js
│   ├── manifest.json        # uni-app 全局配置（App ID、权限等）
│   └── uni.scss             # 全局样式变量
├── static/                  # 静态资源（图片等）
├── package.json
└── README.md
```

## 模式 B：src 内 pages.json（HBuilderX 模版）

```
project-root/
├── src/
│   ├── pages.json           # 页面路由 + tabBar 配置（位置不同于模式 A）
│   ├── pages/
│   │   ├── tab/             # tabBar 页面（主包）
│   │   │   ├── index/
│   │   │   │   └── index.vue
│   │   │   ├── list/
│   │   │   │   └── list.vue
│   │   │   └── user/
│   │   │       └── user.vue
│   │   └── common/          # 公共页面（分包）
│   │       ├── login/
│   │       │   └── index.vue
│   │       └── webview/
│   │           └── index.vue
│   ├── static/
│   ├── components/
│   ├── store/
│   ├── utils/
│   ├── App.vue
│   └── main.js
├── package.json
└── README.md
```

---

## 关键文件说明

### pages.json
- 作用：定义所有页面路径、分包策略、tabBar 配置、全局样式
- 重要字段：
  - `pages[].path`：页面路径（相对于 src/pages/）
  - `tabBar.list[].pagePath`：tabBar 对应页面路径
  - `tabBar.list[].iconPath` / `selectedIconPath`：图标路径
  - `tabBar.list[].text`：tab 文案

### static/tabbar/
- 作用：存放 tabBar 底部导航的图标文件
- 要求：每项需要 2 张图（普通状态 + 选中状态）

### components/
- 作用：存放可复用的自定义组件
- 注意：tabBar 组件通常不放在这里，而是通过 pages.json 配置 + custom-tab-bar 组件实现

### manifest.json
- 作用：uni-app 的 App 全局配置（App 名称、图标、权限列表、平台配置等）

---

## 自定义 tabBar 项目结构

如果使用自定义 tabBar（微信 custom / 支付宝 customize），需要在项目中添加自定义组件：

```
project-root/
├── src/
│   ├── components/
│   │   └── tab-bar/              # 自定义 tabBar 组件
│   │       ├── index.vue
│   │       ├── index.json
│   │       ├── index.wxml
│   │       └── index.wxss
│   ├── pages/
│   │   ├── index/
│   │   │   └── index.vue
│   │   ├── list/
│   │   │   └── list.vue
│   │   └── user/
│   │       └── user.vue
│   └── pages.json
└── package.json
```

---

## 参考项目

| 项目 | GitHub | 说明 |
|------|--------|------|
| vue3-uniapp-template | https://github.com/cshaptx4869/vue3-uniapp-template | 根级 pages.json |
| uniapp-vue3-template | https://github.com/oyjt/uniapp-vue3-template | src 内 pages.json |
| starter-uni | https://github.com/nei1ee/starter-uni | Vue3 + Pinia + UnoCSS |