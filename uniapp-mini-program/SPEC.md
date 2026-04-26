# uni-app 双端小程序开发规范 v1

## 0. 来源分级

| 等级 | 来源 | 用途 |
|------|------|------|
| P0 | uni-app 官方文档 | 路由配置、switchTab 约束、pages.json 注册约束、生命周期 |
| P1 | 微信/支付宝官方文档 | customTabBar 行为、平台 API 限制 |
| P2 | DCloud/GitHub issue | 生态踩坑案例（非官方结论需交叉验证） |

---

## 1. 现象分型

| 类型 | 表现 | 典型根因 |
|------|------|----------|
| A 类 | tabBar 本体闪烁（选中态先错后对） | 选中态更新时序晚于首帧；微信每页实例隔离未同步 |
| B 类 | 内容区抖动（列表/图片加载后撑开） | 异步内容无占位；骨架屏缺失 |
| C 类 | 切换瞬间白屏或明显重绘 | 生命周期重逻辑；多次 setData |
| D 类 | 隐藏/显示 tabBar 过程可见闪现 | 销毁式显隐切换；wx.hideTabBar 误用 |
| E 类 | 支付宝 CE1000 / 红底“元素不存在” | 产物缺页面/组件/JS 依赖；源码别名未编译；风险组件被解析 |
| F 类 | 构建后出现 `pages 2/components 2` | 开发者工具打开了正在删除重建的构建目录 |
| G 类 | 支付返回 toast 显示 `[object Object]` | 支付回调对象被直接转字符串；支付宝 `resultCode` 类型未兼容 |

---

## 2. 根因地图（按出现概率排序）

### G1 [P0-强制] tab 页跳转未走 switchTab
- 官方明确：tabBar 页面只能通过 `uni.switchTab()` 进入
- 证据：`switchTab` API 文档明确"路径后不能带参数"，且目标须是 tabBar.list 中已定义页面
- 触发红线：使用 `navigateTo`/`redirectTo` 进入 tab 页

### G2 [P0-强制] pages.json 未注册或配置缺失
- 官方明确：所有页面须在 pages.json 的 pages 数组中注册，否则不会被打包
- tabBar.list 中的 pagePath 须指向已注册页面
- 证据：pages.json 文档

### G3 [P1-强制] 选中态多源双写
- 官方要求（微信）："如需实现 tab 选中态，要在当前页面下，通过 getTabBar 接口获取组件实例，并调用 setData 更新选中态"
- 禁止：多个位置同时 setData 控制 selectedIndex

### G4 [P1-建议] 布局跳变（CLS）
- 首页图片/banner/card 无占位，数据返回后整体重排
- 骨架屏缺失，首屏内容高度不稳定

### G5 [P1-建议] 生命周期重逻辑
- `onShow` 执行大批量数据重算和批量 setData
- `onLoad` 不会在 tab 切换时重复触发（依赖 onShow 做重初始化会出错）

### G6 [P1-平台差异] 微信/支付宝 customTabBar 机制不同
- 微信：每页独立 tabBar 实例，需在页面侧显式同步
- 支付宝：`customize + overlay` 模式，overlay 属性控制渲染层；某些 API 失效

### G7 [P1-支付宝产物] CE1000 不是源码编译成功即可通过
- 支付宝开发者工具会解析最终产物的 `.json usingComponents`、`.axml` 自定义标签、`.js require()`。
- 构建日志 `DONE` 不等于支付宝工具可打开，必须额外校验页面文件、组件文件、JS 相对依赖。
- `uni_modules` 不能复制源码进产物；源码里的 `@/` 别名、条件编译、ESM import 可能在支付宝工具中 CE1000。
- 需要运行时依赖时优先复制已编译产物，例如从 `dist/build/mp-weixin` 复制 CommonJS 文件，再做平台 API 替换。

### G8 [P1-工具目录] 开发者工具监听构建目录会污染产物
- 不要让支付宝/微信开发者工具打开脚本会 `rm -rf` 后重建的 `dist/build/*` 目录。
- 最终给开发者工具打开独立导出目录，例如 `dist/devtools/mp-alipay-时间戳`。
- 每次重新构建并准备给开发者工具导入时，都必须生成一个全新的目录名；不要覆盖上一次已经被开发者工具打开过的目录。
- 发现 `pages 2`、`components 2`、`static 2`、`uni_modules 2` 时，先关闭工具并清理重复目录，再重建。

### G9 [P1-支付回调] 支付宝支付结果类型与文案
- 支付宝 `requestPayment` 成功回调里的 `resultCode` 可能是字符串 `"9000"`。
- 判断成功必须用 `String(resultCode) === "9000"`。
- `fail/success 非 9000` 返回值是对象时，toast 只能显示 `message/errMsg/memo` 或固定文案，禁止 `String(error)` 直接显示 `[object Object]`。

---

## 3. 强制规则

### R1 路由约束 [P0-强制]
```
✅ MUST: 所有 tab 跳转使用 uni.switchTab({ url: '/pages/xxx/xxx' })
✅ MUST: pages.json 的 tabBar.list 中 pagePath 与实际路径一一对应
✅ MUST: switchTab 的 url 不能带 query 参数
❌ MUST NOT: 使用 navigateTo/redirectTo 进入 tab 页面
❌ MUST NOT: 在 switchTab url 中拼接 ?id=1 等参数
```

### R2 选中态单一来源 [P0-强制]
```
✅ MUST: 选中态只在一处 setData（页面 onShow 或 customTabBar 组件内）
✅ MUST: 微信端在页面 onShow 中通过 getTabBar().setData({ selected: n }) 同步
✅ MUST: 支付宝端在页面 onShow 中通过 my.getTabBar().setData({ selected: n }) 同步
❌ MUST NOT: 页面和组件同时 setData 控制同一个 selectedIndex
❌ MUST NOT: 在 onLoad 中初始化 tab 选中态（onLoad 在 tab 切换时不触发）
```

### R3 避免重复挂载 [P1-强制]
```
✅ MUST: custom tabBar 组件使用 v-show 而非 v-if 控制显隐
✅ MUST: tabBar 组件实例在切换时保留，不强制销毁重建
❌ MUST NOT: 在 onShow 中执行 reMount 操作导致 tabBar 重建
❌ MUST NOT: 使用 hideTabBar/showTabBar 作为常规流程（这是 D 类问题根源）
```

### R4 布局占位 [P1-建议]
```
✅ MUST: 首屏图片设置固定宽高或 aspect-ratio
✅ MUST: 列表首屏骨架屏固定最小高度
✅ MUST: banner/card 占位高度不小于实际内容预期高度
❌ MUST NOT: 完全依赖异步数据返回后撑开布局（CLS 根源）
```

### R5 生命周期减载 [P1-建议]
```
✅ MUST: onShow 中的重任务拆分为异步分片
✅ MUST: 大批量 setData 合并为单次调用
✅ MUST: 非首屏必要数据延迟到 onReady 之后
❌ MUST NOT: 在 onShow 中执行无必要的全量数据重新请求
❌ MUST NOT: 短时间内多次触发 setData（超过 2 次/切换视为异常）
```

### R6 支付宝产物完整性 [P1-强制]
```
✅ MUST: 最终 app.json 声明页面都存在 .json/.js/.axml
✅ MUST: 所有 usingComponents 指向的组件存在 .json/.js/.axml
✅ MUST: 所有 JS 相对 require() 目标文件存在
✅ MUST: 最终产物 JS 不含 from "@/..."、require("@/...")、import("@/...")
✅ MUST: 轻页面优先用原生 view/text/button，避免为少量图标引入 Layout/NavBar/uv-icon
❌ MUST NOT: 把源码 uni_modules 直接复制进支付宝最终产物
❌ MUST NOT: 只凭 uni build 成功判断可提审
```

### R7 开发者工具目录隔离 [P1-强制]
```
✅ MUST: 构建目录 dist/build/* 只给脚本使用
✅ MUST: 开发者工具打开 dist/devtools/* 或项目根目录中稳定指向产物的配置
✅ MUST: 每次给支付宝开发者工具导入都使用新目录名，例如 dist/devtools/mp-alipay-YYYYMMDD-HHmmss
✅ MUST: 构建前后扫描并清理/拦截 "* 2"、"* 3" 重复目录
❌ MUST NOT: 开发者工具打开正在被脚本删除重建的 dist/build/* 目录
❌ MUST NOT: 覆盖开发者工具已经打开过的导入目录
```

### R8 支付宝支付回调 [P1-强制]
```
✅ MUST: String(resultCode) === "9000" 才视为成功
✅ MUST: 失败对象转成明确文案：message/errMsg/memo/fallback
❌ MUST NOT: uni.showToast({ title: String(error) }) 直接展示对象
```

---

## 4. 双端差异处理

### 4.1 微信小程序 [P1]

**custom tabBar 关键机制：**
- 每个 tab 页下的 custom tabBar 组件实例**相互隔离**，不能跨页共享同一个实例
- 必须在**当前页面**调用 `this.getTabBar()` 获取实例，再调用 `setData({ selected: n })` 更新选中态
- `wx.setTabBarItem` 等样式类 API 在 custom 模式下**失效**
- Skyline 模式下 `getTabBar` 是**异步回调**，需等待回调完成后再 setData

**微信端 MUST DO：**
```js
// 页面 onShow 中同步选中态
onShow() {
  if (typeof this.getTabBar === 'function' && this.getTabBar) {
    this.getTabBar((tabBar) => {
      tabBar.setData({ selected: 0 })  // 0=首页, 1=列表, 2=我的
    })
  }
}
```

**微信端 MUST NOT：**
- ❌ 跨页面共享一个 tabBar 实例来控制状态
- ❌ 在 customTabBar 组件的 created/mounted 中等待父组件传递 selectedIndex（时序问题）

### 4.2 支付宝小程序 [P1]

**customize tabBar 关键机制：**
- `tabBar.customize: true` 开启自定义模式
- `tabBar.overlay: true` 开启 Native 渲染模式（两个都必须为 true）
- `my.hideTabBar()` 在 customize 模式下**失效**
- `this.getTabBar()` 返回实例上有 `overlay` 属性（boolean），标识当前渲染模式
- 若条件不满足，会自动降级为普通自定义 tabBar

**支付宝端 MUST DO：**
```js
onShow() {
  const tabBar = this.getTabBar()
  if (tabBar) {
    tabBar.setData({ selected: 0 })
    // 可通过 tabBar.overlay 判断当前是否为 overlay 模式
    console.log('overlay mode:', tabBar.overlay)
  }
}
```

**支付宝端 MUST NOT：**
- ❌ 依赖 `my.hideTabBar()` 控制 tabBar 显隐（失效）
- ❌ 只配置 `overlay: true` 而未配置 `customize: true`（无效）

### 4.3 双端差异汇总

| 行为 | 微信 custom | 支付宝 customize |
|------|------------|----------------|
| 实例隔离 | ✅ 每页独立 | ✅ 每页独立 |
| 状态同步方式 | getTabBar().setData() | getTabBar().setData() |
| 样式 API 失效 | wx.setTabBarItem 等 | my.hideTabBar 等 |
| overlay 模式 | 无（custom 配置） | customize + overlay 同时为 true |
| Skyline 异步回调 | ✅ 是 | 支付宝无 Skyline 模式 |
| 支付 resultCode | 微信无此码 | 可能是字符串 `"9000"` |
| 产物解析 | 微信容忍度较高 | 对 usingComponents/AXML/require 更敏感 |

---

## 5. 验收标准

- [ ] 连续快速点击 20 次 tab，视觉无明显闪烁或错位
- [ ] 首次进入每个 tab 页均不出现选中态错乱
- [ ] 弱网下切换 tab，内容区域不发生突兀位移
- [ ] 微信与支付宝真机表现一致或差异可解释且已记录
- [ ] 所有 tab 跳转均通过 switchTab（代码扫描红线通过）
