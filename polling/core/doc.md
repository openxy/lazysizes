# 轮询核心定形

## 本版本引入的概念

**懒加载（lazy loading）**：图片、iframe 先不加载。真实地址放在 `data-src` 里，元素进入视口附近才换给 `src`。首屏快、流量省。

**事件轮询**：本案例（polling）的核心机制，也是 2016 年前所有懒加载库的共同形态——监听 scroll/resize 等一大排事件，每次触发节流后**全量扫描**所有带 `lazyload` 类的元素，用 `getBoundingClientRect` 判断谁进了「视口 + 预载环」，谁进谁加载。IntersectionObserver 出现之前，浏览器没有「元素进入视口」的通知机制，只能反复问。

**节流（throttle）**：滚动一秒触发几十次事件，不能每次都扫描。throttle 保证两次检查至少间隔约 99ms，且执行对齐到下一帧（`lazysizes.js:99-128`）。

这是 polling 案例的链首版本：整个经典实现一次看全，后两版只做增量修补。

## 怎么学这个版本

1. 跑 `index.html`（运行区），滚动 + Network 面板，看图片是否滚到附近才发请求；
2. 深读 `lazysizes.js`（667 行）：文件级逐段解读在本案例末版 polling/umd-split 的 doc/ 下（按 1.5.0 行号写，机制在 1.3.1 已全部存在，行号有 ±20 行漂移，先用「怎么学」第 1 步的运行观察建立直觉再读）。

## 关键机制速览

- **事件面挂得很宽**：scroll/resize 之外还有 focus、mouseover、click、transitionend、animationend、hashchange，加 MutationObserver 盯 DOM 增删——任何可能改变可见性的入口都当触发器，反正是节流的全量扫描，多几个信号只赚不亏；
- **活集合**：`getElementsByClassName` 返回的集合随 DOM 自动更新，动态插入的懒加载元素天然被发现；
- **expand 预载环**：视口四边外扩一圈，元素进环就提前加载——用户还没滚到，图已就位；
- **loadMode 档位**：页面加载完 3 档（激进预载），用户一滚动降 2 档，停滚再升回——滚动中省流量、停滚后补预载；
- **注释里的下一代草稿**（`lazysizes.js:131` 起）：requestIdleCallback 版 throttle 整段写好、注释放着——rIC 当时只有 Chrome 有。这段草稿要到上游 2.0.0 才转正（mature 案例第一版），跨越多年。

## 局限（本案例教学的出发点）

- 每次检查全量扫描：未加载元素挨个 `getBoundingClientRect`（强制布局），元素上千时滚动会卡；
- scroll 事件只说明「可能有什么变了」，与「元素进入视口」没有直接关系——这个根本缺陷正是 IntersectionObserver 要解的题（mature 案例末版的 src/ 实验变体里有完整答案）。

## 思考题

- 页面有 2000 张懒加载图时，一次 `checkElements` 的开销在哪个量级？瓶颈是 JS 计算还是布局？
- 为什么监听要挂在 window 上还加 `true`（捕获阶段）？
- 若把 expand 设为 0，用户体验和流量各怎么变？
