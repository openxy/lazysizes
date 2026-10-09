# 精读：lazysizes.js（1.5.0，696 行——事件轮询懒加载的完整实现）

发布产物本身未压缩、可读。整文件是一个 UMD 包装的 IIFE（模块同时支持 script 标签和 CommonJS 引入），内部按「工具层 → loader 主体 → autoSizer → 配置」分四段。建议按下面的段落顺序读，行号指本版文件。

## 一、UMD 头与运行环境（1-33 行）

- 1-7 行：UMD（Universal Module Definition，通用模块定义）标准壳——浏览器直接当全局用，构建工具当模块导入；
- 10 行：`if(!document.getElementsByClassName){return;}`——老浏览器直接放弃，库的第一行就是兼容性声明；
- 16 行：探测 `HTMLPictureElement`（picture 元素支持），决定后面要不要走 polyfill（补丁层）分支；
- 26 行：`var rAF = window.requestAnimationFrame || setTimeout`——能力探测加回退，老技巧。

## 二、工具层（36-182 行）

- 36-54 行：hasClass/addClass/removeClass——**手写 className 正则匹配**，正则按类名缓存在 `regClassCache`。为什么不用 classList？当年 IE9/10 的 classList 有性能与兼容问题，库的定位是「零依赖跑遍所有浏览器」；
- 66-73 行：triggerEvent 用 `createEvent('CustomEvent')` 派发自定义事件——库的扩展点全靠它（`lazybeforeunveil` 等事件让插件能拦截加载流程）；
- 84-97 行：getCSS/getWidth——autoSizer 用它算图片显示宽度，宽度太小就沿父链上溯（见 doc.md 概念里的 minSize）；
- 99-128 行：**throttle 节流**，本版调度核心。读法：
  - `running` 标志保证同一时间只有一个排队中的执行；
  - `125 - (Date.now() - lastTime)` 算距上次执行的剩余窗口，不足 6ms 就抬到 6ms；
  - 到点后先 `rAF`，rAF 回调里再 `setTimeout` 才真正 run——rAF 对齐下一帧（保证布局之后、绘制之前），setTimeout 再推后一个任务队列，躲开帧内拥挤；
- 130-182 行：**整段注释掉的 rIC 版 throttle**（requestIdleCallback 空闲回调版，参数更细：66ms 窗口、优先调用阈值 22ms、可传 isPriority 提级）。2015 年 rIC 只有 Chrome 有，作者写好了等普及——这段注释在上游 2.0.0 转正（mature 案例第一版的 diff 里能看到它从注释变成正文），是本案例集跨案例对照的一条暗线。

## 三、loader 主体（184-574 行，最核心）

闭包内的状态：`lazyloadElems`（全部待加载元素）、`isLoading`（在途加载计数）、`loadMode`（预载档位）、`currentExpand`（当前预载环半径）。

- 202-211 行：resetPreloading——每张图 load/error 事件都会让在途计数减一，2.5 秒兜底超时防计数泄漏；
- 213-237 行：isNestedVisible——元素矩形进了环还不够，还要沿 offsetParent 链确认没有 `opacity:0` 的祖先、没有被 `overflow:hidden` 的祖先裁掉。**防的是「矩形在环内但实际看不见」的误加载**；
- 239-306 行：**checkElements 主循环**，轮询的心脏：
  - 248-255 行：首次运行时按屏幕尺寸定 expand（大屏 550/410，小屏 359），乘 expFactor(1.7) 得预载环；
  - 257-264 行：loadMode 档位切换——空闲且滚了好几轮用 preloadExpand（大环），普通滚动用 defaultExpand，忙时收缩到 0；
  - 266-300 行：遍历所有待加载元素，读 `getBoundingClientRect`，四边与环比较（`eLbottom >= elemNegativeExpand` 等四条），命中且 isNestedVisible 通过就 `unveilElement`。注意 289 行 `(eLbottom || eLright || eLleft || eLtop)`——display:none 的元素四边全 0，靠这条件排除；
  - 293 行：`if(isLoading > 9){break;}`——一次最多让 10 张图在途，避免带宽挤爆首屏；
- 310-320 行：类切换状态机（lazyload → lazyloading → lazyloaded），CSS 靠这三个类做淡入特效，加载状态机与视觉状态机同构；
- 351-372 行：rafBatch——rAF 批队列，把多个 unveil 合到一帧执行；
- 374-465 行：**unveilElement 揭幕**：先派 `lazybeforeunveil`（插件拦截点，defaultPrevented 就放弃）→ 处理 sizes/srcset/src 的 data- 前缀换名 → img 直接赋 src，iframe 走 `contentWindow.location.replace`（保历史记录干净，跨域时回退直接赋值）→ rAF 后收尾换类；
- 467-497 行：onload——window.load 后进「完成态」，loadMode 升 3（激进）；之后每次 scroll 先降 2、停滚 99ms 后由 afterScroll 升回 3。**滚动中省流量、停滚后补预载**；
- 499-532 行：又一段注释——防抖（debounce，间隔 99ms 合并）版 onload，同样在 4.x 转正；
- 534-570 行：初始化。538-539 行两个 `getElementsByClassName` 拿**活集合**（DOM 增删自动反映，动态插入的懒加载元素天然被发现）；542-559 行把 throttledCheckElements 挂上全部事件（见 doc.md 概念）；546-552 行 MutationObserver 盯 DOM 变化，没有就用 DOMNodeInserted + setInterval(999ms) 兜底；561-567 行按 readyState 决定立即 onload 还是等 load 事件（`/d$|^c/` 匹配 "complete"/"interactive" 的尾或头字符），另设 20 秒兜底。

## 四、autoSizer（577-633 行）

`data-sizes="auto"` 的实现：懒加载场景下 `sizes` 属性（告诉浏览器图片将显示多宽，供 srcset 挑图）经常写不准，这模块在揭幕前用 `elem.offsetWidth` 实测宽度（小于 minSize=40 就沿父链上溯），写成像素值塞给 sizes。resize 时节流重算。

## 五、配置与启动（643-694 行）

- 646-664 行：默认配置对象（类名、data- 属性名、expand/expFactor/hFac/loadMode 等）；
- 666-674 行：与 `window.lazySizesConfig` 浅合并——**用户在引入库之前设全局变量就能改默认行为**，无依赖库的经典配置法；
- 676-680 行：`setTimeout(...)` 异步启动，给 DOM 一点喘息；`init: false` 可完全接管启动时机；
- 683-694 行：导出 API（cfg/loader/autoSizer/fire 等），插件的挂载面。
