# scroll 事件轮询懒加载

## 本版本引入的概念

**懒加载（lazy loading）**：图片、iframe 先不加载。真实地址放在 `data-src` 里，元素进入视口附近才把它换给 `src`。首屏快、流量省。

**事件轮询**：本版的核心机制，也是 2016 年前所有懒加载库的共同形态——监听 scroll/resize 等一大排事件，每次触发都做一次节流，然后**全量扫描**所有带 `lazyload` 类的元素，用 `getBoundingClientRect` 判断谁进入了「视口 + 预载环」，谁进谁加载。IntersectionObserver 出现之前，浏览器没有「元素进入视口」的通知机制，只能这么反复问。

**节流（throttle）**：滚动一秒能触发几十次事件，不能每次都扫描。`lazysizes.js` 里的 throttle 保证两次检查至少间隔约 125ms，且实际执行对齐到下一帧（`lazysizes.js:99-128`）。

## 怎么学这个版本

1. 先跑 `index.html`（运行区），滚页面、开 Network 面板，看图片是不是滚到附近才发请求；再看右侧文档「用法面」小节对照 HTML 写法；
2. 然后按「读码路线」精读 `lazysizes.js`（文件级解读在本版 doc/ 下）——这是本案例集最重要的一个文件，值得逐段过。

## 关键机制（细节在文件级解读里）

- **事件面挂得很宽**（`lazysizes.js:542-559`）：scroll/resize 之外还有 focus、mouseover、click、transitionend、animationend、hashchange，加一个 MutationObserver 盯 DOM 增删。逻辑是「任何可能改变可见性的入口都当触发器，反正是节流的全量扫描，多来几次信号只赚不亏」。
- **活集合**（`lazysizes.js:538`）：`getElementsByClassName` 返回的集合随 DOM 自动更新，新插入的懒加载元素天然被发现，不需要登记。
- **expand 预载环**（`lazysizes.js:248-264`）：视口四边外扩一圈（按屏幕尺寸 359~550px），元素进环就提前加载——用户还没滚到，图已就位。
- **loadMode 档位**（`lazysizes.js:474-496`）：页面加载完是 3 档（激进预载），用户一开始滚就降 2 档（只用默认环），省流量。
- **两段被注释的代码**（`lazysizes.js:130-182`、`499-532`）：requestIdleCallback 版 throttle 和防抖版 onload——作者在 1.x 就写好了下一代调度，注释留着等浏览器普及。v4 系列读到的就是它们转正后的样子，这是个难得的「代码考古」对照点。

## 局限

- 每次检查是全量扫描：未加载元素挨个 `getBoundingClientRect`（强制布局），元素上千时滚动会卡；
- scroll 事件只说明「可能有什么变了」，与「元素进入视口」没有直接关系，所以才需要反复核对——这个根本缺陷正是后来 IntersectionObserver 要解的题（见 v4 系列里的实验变体）。

## 思考题

- throttle 里 rAF 套 setTimeout（`lazysizes.js:108-113`），为什么不是直接 rAF 或直接 setTimeout？
- `isNestedVisible`（`lazysizes.js:213-237`）为什么要沿祖先链查 opacity 和 overflow？删掉它会出现什么误加载？
- 页面有 2000 张懒加载图时，一次 `checkElements` 的开销在哪个量级？瓶颈是 JS 计算还是布局？
