# tabBar 闪动问题快速排查手册（15 分钟版）

## Step 1：判断问题类型（3 分钟）

| 问题类型 | 判断方法 |
|----------|----------|
| A 类 tabBar 本体闪 | 慢速点击一次，观察 tabBar 图标/下划线是否先错后对 |
| B 类内容区抖 | 快速点击，观察列表/banner 是否整体位移 |
| C 类白屏重绘 | 切换时观察是否有一瞬间全白 |
| D 类显隐闪现 | 观察 hide/show tabBar 过程是否有闪现 |

**记录**：本次问题属于 ___ 类（填 A/B/C/D）

---

## Step 2：路由路径检查（3 分钟）

### 检查 pages.json
打开 pages.json，检查：
- [ ] `tabBar.list` 中是否包含所有 tab 页面
- [ ] 每个 item 的 `pagePath` 是否与 pages 数组中的路径完全一致

### 全局搜索红线
```bash
# 在项目中搜索所有非 switchTab 的 tabBar 跳转
# 如果返回任何结果，说明有红线问题
grep -rn "navigateTo\|redirectTo" --include="*.vue" --include="*.js" | grep -i "tab\|pages/"
```

**结论**：路由路径是否规范？___

---

## Step 3：选中态更新频率打点（3 分钟）

在 `onShow` 中加入计数日志：

```js
onShow() {
  console.log('[tabCheck] onShow triggered', Date.now())
  // 检查 setData 调用次数（可通过调试工具或全局代理实现）
}
```

**判断**：
- 单次切换 `onShow` 被调用 >2 次 → 有重复调用问题
- 单次切换 `setData` >2 次 → 需要合并

**结论**：选中态更新频率是否正常？___

---

## Step 4：首屏占位验证（3 分钟）

临时给所有首屏图片/card/banner 设置固定占位：

```css
/* 临时方案：所有首屏图片强制占位 */
img, .banner, .card {
  min-height: 100px;
  background: #f0f0f0;  /* 占位背景 */
}
/* 或 */
.aspect-ratio-box {
  aspect-ratio: 16/9;
}
```

切换 tab 观察是否消除"跳一下"。

**结论**：占位修复是否有效？___

---

## Step 5：生命周期抖动确认（3 分钟）

暂时注释掉 `onShow` 中的重逻辑：

```js
// onShow() {
//   await fetchData()   // 暂时注释
//   this.processHeavy() // 暂时注释
// }
```

观察切换是否平稳。

**结论**：是否为生命周期抖动？___

---

## Step 6：双端真机复测（结果汇总）

| 平台 | 现象 | 根因归类 |
|------|------|----------|
| 微信开发者工具 |  |  |
| 微信真机 |  |  |
| 支付宝开发者工具 |  |  |
| 支付宝真机 |  |  |

**差异处理**：
- 若双端表现不一致 → 对照 SPEC.md 4.3 节双端差异汇总表排查
- 若模拟器 ≠ 真机（尤其是支付宝）→ 以真机为准

---

## 问题类型 → 优先修复映射

| 问题类型 | 优先修复方向 |
|----------|-------------|
| A 类 | 检查 R2 选中态单一来源 + 微信/支付宝 getTabBar 同步 |
| B 类 | 执行 R4 布局占位 |
| C 类 | 执行 R5 生命周期减载 + R3 避免重复挂载 |
| D 类 | 禁止常规流程使用 hideTabBar/showTabBar |
| 混合型 | 按 A→B→C→D 顺序依次排查 |