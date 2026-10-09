# SVG 支持与调度微调

## 本版本引入的概念

**一手兼容 SVG**：上一版类操作直接读写 `ele.className`——SVG 元素的 `className` 不是字符串，是 `SVGAnimatedString` 对象，字符串方法全不可用。这版换成 `getAttribute('class')` / `setAttribute('class', ...)`（`lazysizes.js:42-54`），一个属性访问路径同时吃 HTML 和 SVG。**没有 if 分支、没有特性检测，换 API 就是兼容**——比「检测分支」高一个段位的写法。

**hFac 横向系数**（1.3.2 引入）：预载环的水平半径单独乘一个系数（默认 0.8）。横向滚动场景（画廊、表格）与纵向滚动对「提前量」的需求不同，环从正方形变长方形。

**调度参数打磨**：throttle 的空闲窗口 99ms 收紧到 66ms、加快速调用阈值 22ms；`setImmediate` 换 `setTimeout`（`lazysizes.js:145`，setImmediate 只在 IE/Edge 有且行为怪）。参数级演进——不换机制，调手感。

## 本版本的改动（差异面板对照上一版）

核心文件 `lazysizes.js` 约 74 行变动，集中在四处：

- 类操作三件套改 getAttribute/setAttribute（上面已讲，点开差异面板第一个块就是它）；
- throttle 参数与 rIC 草稿注释同步更新（草稿仍在注释里，还没转正）；
- `isNestedVisible`（`lazysizes.js:213-237` 一带）：加 body 可见性短路判断、offsetParent 循环收紧——修「祖先链上有 overflow 裁剪时误判可见」的边角；
- 插件侧（bgset/parentFit）减少布局抖动（layout thrashing：读写交替强制浏览器反复重排，改成先集中读再集中写）。

## 思考题

- `SVGAnimatedString` 的 `baseVal`/`animVal` 各是什么？为什么框架的 class 工具基本都用 setAttribute？
- hFac 为什么只调横向不调纵向？（提示：网页的主滚动轴是哪个，用户预判滚动的方向感）
- 「减少布局抖动」为什么对懒加载库特别重要？（提示：它本来就在滚动路径上高频跑）
