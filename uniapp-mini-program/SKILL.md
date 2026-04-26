---
name: uniapp-mini-program
description: Use when diagnosing or implementing uni-app mini-programs, especially WeChat/Alipay tabBar behavior, pages.json and switchTab rules, lifecycle issues, Alipay CE1000/red-screen component errors, devtools duplicate output folders, uni_modules runtime dependency problems, and Alipay requestPayment resultCode/toast handling.
---

# uniapp-mini-program skill

## 触发词
uni-app, uniapp, tabBar 闪, tabBar 抖, tabBar 闪烁, tabBar 抖动,
小程序 tabBar, switchTab, pages.json, onShow onHide, 自定义 tabBar,
微信 tabBar, 支付宝 tabBar, tab 切换, tabBar flash, tabBar jitter,
支付宝小程序, 支付宝红码, CE1000, 元素不存在, 组件不存在,
dist/build, pages 2, components 2, uni_modules, requestPayment, resultCode

## 核心能力
1. tabBar 闪动问题分类（A/B/C/D 四类）
2. 根因定位（6 大根因 + 强制规则 R1-R5）
3. 双端差异处理（微信 custom vs 支付宝 customize）
4. 快速排查（15 分钟版 PLAYBOOK）
5. 代码评审红线检查（CHECKLIST）
6. 验收标准确认
7. 支付宝 CE1000 / 红码产物排查
8. 支付宝开发者工具导入目录隔离
9. uni_modules 编译产物依赖校验
10. 支付宝支付回调 resultCode / toast 对象处理
11. 每次构建导出全新开发者工具目录，避免覆盖已打开目录

## 规则优先级
- [P0] 官方强制约束：pages.json 注册、switchTab 限制、tabBar 配置要求
- [P1] 平台强制约束：微信/支付宝 customTabBar 行为差异
- [P1] 支付宝工具约束：最终产物不得含源码别名、缺失组件、重复 `* 2` 目录
- [P1] 支付回调约束：支付宝 `resultCode` 按字符串处理，toast 不得直接显示对象
- [P2] 建议性实践：布局占位、生命周期减载

## 文件关联
- SPEC.md          → 详细规范条文与双端差异
- PLAYBOOK.md      → 排查步骤操作手册
- CHECKLIST.md     → 代码扫描清单
- REFS/*.md        → 官方文档引用摘要（供 agent 查证）

## 关键路径约定
项目结构（任选其一）：
A. 根级 pages.json：/pages.json, /src/pages/, /src/components/
B. src 内 pages.json：/src/pages.json, /src/pages/, /src/components/
tabBar 配置：pages.json 或 src/pages.json 的 tabBar.list
自定义 tabBar 组件：src/components/tab-bar/（需自行创建）

## 触发时行为
1. 命中触发词 → 加载本 skill
2. 识别问题类型（A/B/C/D）→ 调用 SPEC.md 分型规则
3. 匹配根因 → 执行对应 R 规则检查
4. 跨端问题 → 参考 REFS/ 双端差异文档
5. CE1000 / 红码 / 支付异常 → 先执行 CHECKLIST.md 的支付宝产物红线检查
6. 完整排查 → 参考 PLAYBOOK.md 步骤执行
