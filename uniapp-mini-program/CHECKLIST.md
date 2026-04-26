# 代码评审红线清单

## R1 路由约束 [P0]
- [ ] pages.json 中 tabBar.list 包含所有 tab 页的 pagePath
- [ ] 全局搜索确认无 `navigateTo`/`redirectTo` 进入 tab 页路径
- [ ] 所有 tabBar 跳转均使用 `uni.switchTab()`
- [ ] switchTab 的 url 不包含 ?query 参数

**检测命令**：
```bash
grep -rn "navigateTo\|redirectTo" --include="*.vue" --include="*.js" | grep -i "pages/index\|pages/list\|pages/user"
```

## R2 选中态单一来源 [P0]
- [ ] 搜索 `selected` 相关 setData，确认只有一处
- [ ] 微信端：页面 onShow 中通过 `getTabBar().setData()` 同步
- [ ] 支付宝端：页面 onShow 中通过 `my.getTabBar().setData()` 同步
- [ ] 不存在页面和组件同时维护 selectedIndex 的情况

**检测命令**：
```bash
grep -rn "selected\|setTabBar" --include="*.vue" --include="*.js"
```

## R3 避免重复挂载 [P1]
- [ ] tabBar 组件使用 v-show，不使用 v-if 做显隐
- [ ] onShow 中无 `uni.hideTabBar`/`uni.showTabBar` 调用
- [ ] 无业务逻辑强制重建 tabBar 组件

**检测命令**：
```bash
grep -rn "hideTabBar\|showTabBar\|v-if.*tabBar\|reMount" --include="*.vue"
```

## R4 布局占位 [P1]
- [ ] 首屏 img 有 mode="widthFix" 或固定宽高
- [ ] 列表页首屏有骨架屏或最小高度占位
- [ ] banner/card 有固定占位高度

**检测命令**：
```bash
grep -rn "aspect-ratio\|min-height\|skeleton\|placeholder" --include="*.vue" --include="*.scss" --include="*.css"
```

## R5 生命周期减载 [P1]
- [ ] onShow 中无非必要的 await fetchData（全量数据重请求）
- [ ] 大批量 setData 已合并为单次
- [ ] 耗时操作已拆分到下一个任务帧

**检测命令**：
```bash
# 人工审查 onShow 中的逻辑复杂度
grep -rn "onShow\|onLoad" --include="*.vue" | head -30

# 检测短时间内多次 setData
grep -rn "setData" --include="*.vue" --include="*.js"
```

## 双端差异 [P1]
- [ ] 微信端 custom tabBar 在页面侧显式同步（getTabBar().setData）
- [ ] 支付宝端已真机验证（模拟器不等于真机）
- [ ] 无平台特有 API 混用（如 wx.* 在支付宝中或 my.* 在微信中）

**跨平台 API 检测**：
```bash
# 检测微信 API 误用
grep -rn "wx\." --include="*.vue" --include="*.js" | grep -v "wx.request\|wx.getStorage"

# 检测支付宝 API 误用
grep -rn "my\." --include="*.vue" --include="*.js" | grep -v "my.request\|my.getStorage"
```

## 支付宝产物红线 [P1]
- [ ] 最终导入目录不是正在重建的 `dist/build/mp-alipay`
- [ ] 每次构建后给开发者工具导入的是全新目录名，例如 `dist/devtools/mp-alipay-YYYYMMDD-HHmmss`
- [ ] 没有覆盖开发者工具已经打开过的旧导入目录
- [ ] 最终导入目录不存在 `pages 2/components 2/static 2/uni_modules 2`
- [ ] `app.json` 声明页面均有 `.json/.js/.axml`
- [ ] 所有 `usingComponents` 均能解析到 `.json/.js/.axml`
- [ ] 所有 JS 相对 `require()` 目标存在
- [ ] 最终产物 JS 不含 `@/` 源码别名
- [ ] 红底“元素不存在”页面的 `.json usingComponents` 已确认；轻页面不依赖 `Layout/NavBar/uv-icon`

**检测命令**：
```bash
# 重复目录
find dist -name '* 2*' -o -name '* 3*' -o -name '* 4*'

# 源码别名残留
grep -R "from ['\"]@/\\|require(['\"]@/\\|import(['\"]@/" dist/devtools/mp-alipay-* dist/build/mp-alipay

# JS 相对 require 缺失检查
node - <<'NODE'
const fs = require('fs'), path = require('path')
const root = process.argv[1] || 'dist/build/mp-alipay'
function walk(d){return fs.existsSync(d)?fs.readdirSync(d,{withFileTypes:true}).flatMap(e=>{const f=path.join(d,e.name);return e.isDirectory()?walk(f):e.isFile()?[f]:[]}):[]}
let missing=[]
for(const jsFile of walk(root).filter(f=>f.endsWith('.js'))){
  const src=fs.readFileSync(jsFile,'utf8')
  for(const m of src.matchAll(/require\(["']([^"']+)["']\)/g)){
    if(!m[1].startsWith('.')) continue
    const target=path.resolve(path.dirname(jsFile),m[1])
    if(!fs.existsSync(target)) missing.push(`${path.relative(root,jsFile)} -> ${m[1]} => ${path.relative(root,target)}`)
  }
}
console.log(missing.join('\n') || 'REFERENCES_OK')
NODE
```

## 支付宝支付回调 [P1]
- [ ] `requestPayment` 成功判断使用 `String(resultCode) === "9000"`
- [ ] fail 或非 9000 分支只 reject 字符串文案，不 reject 原始对象给 toast
- [ ] 页面 catch 中不使用 `String(error)` 直接显示对象

**检测命令**：
```bash
grep -R "resultCode === 9000\\|String(error\\|String(.*error" --include="*.vue" --include="*.js" .
```

---

## 评审结论模板

```
项目：_________________
评审人：_________________
日期：_________________

| 红线 | 状态 | 文件位置 | 备注 |
|------|------|----------|------|
| R1 路由 | ✅通过/❌违规 |  |  |
| R2 选中态 | ✅通过/❌违规 |  |  |
| R3 重复挂载 | ✅通过/❌违规 |  |  |
| R4 布局占位 | ✅通过/❌违规 |  |  |
| R5 生命周期 | ✅通过/❌违规 |  |  |
| 双端差异 | ✅通过/❌违规 |  |  |
| 支付宝产物 | ✅通过/❌违规 |  |  |
| 支付回调 | ✅通过/❌违规 |  |  |

总体结论：✅ 可发布 / ❌ 需修复后重新评审
```
