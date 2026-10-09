# UMD 拆分与后台标签页

## 本版本引入的概念

**AMD 拆分**：上一版文件头同时注册 CommonJS 和 AMD（`define(lazySizes)`）。问题：AMD 注册器拿到的是**已经执行完的实例**，插件想 `define(['lazysizes'], ...)` 按依赖注入就拿不到加载时机。这版把 AMD 注册拆到独立的 `lazysizes-umd.js`，主文件只留 CommonJS 和全局——**一个文件伺候不好两种模块生态时，拆成两个文件各伺候各的**（破坏性变更，主版本号因此从 1.4 跳 1.5）。

**后台标签页治理**：标签页切到后台时 rAF 停摆、定时器节流，但检查还在排队。这版让 onload 后的首轮检查在 `document.hidden` 时绕过节流直接跑（`lazysizes.js:483-488`）——后台页面攒着不动反而错过后台预载的窗口期。

## 本版本的改动（差异面板对照上一版）

- `lazysizes.js` 头部模块注册拆分（文件头差异块）+ 新文件 `lazysizes-umd.js`；
- 后台标签页：`document.hidden` 分支直跑 `checkElements`；
- `rafSwitchLoadingClass` 的 rAF 包装（类切换进帧对齐，避免加载完成的样式跳变）；
- 修 Firefox 过早挂 lazyloaded 类的竞态。

本案例（polling）到这里收官：轮询架构下能打磨的手感基本打磨完。注意 `lazysizes.js:130-182` 和 `499-532` 那两段注释——rIC 调度与防抖的**完整草稿仍然躺着**。它们是下一案例（mature）第一版的正文：跨案例边界处，正好用「读注释猜未来」收尾。

## 怎么学这个版本

本版是 polling 案例的定版形态，文件级解读按它写：`doc/lazysizes.js.md`（667→696 行逐段精读）、`doc/index.html.md`（官方演示页 = 用法说明书：LQIP 低清占位、响应式 data-srcset、picture、iframe、data-expand 特效、动态插入自动发现）。

## 思考题

- AMD 的 `define` 和 CommonJS 的 `module.exports` 在**加载时机**上差在哪，为什么这个差异让「一个文件双注册」行不通？
- 后台标签页里 rAF 停摆、setTimeout 被限到每秒 1 次——如果你写轮询逻辑，怎么测试这种环境？（提示：Chrome DevTools 的后台节流模拟）
- 1.5.0 是「Breaking change」却只升了小版本号（1.4→1.5）。对比现在主流 semver 实践，说明什么？
