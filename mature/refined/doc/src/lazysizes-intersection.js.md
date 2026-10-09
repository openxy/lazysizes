# 精读：src/lazysizes-intersection.js（540 行——IntersectionObserver 实验变体，从未转正）

主产物 `lazysizes.js` 的同胞实验：**同一个库、同一种用法，loader 内部换掉了发动机**。它没有被构建进根目录产物，只活在 src/ 里。读它最大的价值是对照——事件轮询架构里的每个部件，在观察者架构里都有一对应物或者直接消失。

## 一、开场即自白（13 行）

```js
if(!window.IntersectionObserver || !document.getElementsByClassName || !window.MutationObserver){return;}
```

要求 IO + MutationObserver 双支持，缺一个整库静默退出——**无回退**。这就是它进不了主产物的根本原因（详见版本文档）。

## 二、发动机换新：扫描没了

对照主产物的 checkElements 全量循环，这里只剩两个观察器（396-409 行）：

- `inviewObserver = new IntersectionObserver(unveilElements)`——无 rootMargin，元素**真正进入视口**才回调，回调里直接揭幕；
- `preloadObserver = new IntersectionObserver(lazyUnveilElement, {rootMargin: expand + 'px ' + expand*hFac + 'px'})`——rootMargin 把观察边界外扩，**预载环的概念从「每次扫描现算矩形」变成「告诉浏览器替我盯着这个圈」**。

滚动监听、resize 监听、checkElements 循环、loadMode 档位状态机、throttle——全部消失。浏览器只在你关心的事发生时叫你一次，**拉（轮询）变推（通知）**，这是两种编程模型的分水岭。

## 三、那扫描消失后，谁来发现新元素？（373-394 行）

`new MutationObserver(addElements).observe(docElem, {childList: true, subtree: true, attributes: true})`——DOM 一变就遍历活集合，给新元素挂 `observe`。注意 376-377 行的 `_lazyAdd` 标记（去重）和 359-371 行的 removeLazyClass：新元素登记后**立刻把 lazyload 类换成 lazyWaitClass**（364-365 行）——活集合是按类名查的，换类等于把它从「待处理池」里移出，MutationObserver 再触发时就不会重复处理。**用类的更替当登记簿**，小而巧。

## 四、揭幕后的反注册（320-325 行）

`inviewObserver.unobserve(elem); preloadObserver.unobserve(elem)`——已加载的元素从观察器摘除，观察成本只覆盖未加载元素，越用越轻。对照轮询版：活集合里的元素加载后只是加了 lazyloaded 类，扫描时仍会被 `hasClass` 跳过——**每轮都在为已完成的元素付判断成本**。

## 五、预载的并发会计（191-198、341-355 行）

preloadObserver 命中时不直接加载，进 `lazyQuedElements` 队列，由在途计数（isLoading/isPreloadLoading）控制逐个放出——预载不能挤占真实需求，观察者架构照样需要应用层的并发管理。**底层模型的先进不消掉上层的调度问题**。

## 六、其他可见的代际差

- 39-49 行：类操作直接用 `classList`——这变体不伺候老浏览器，代码立刻干净一截；
- 490-508 行：配置表少了 loadMode/throttleDelay 等（用不上了），多了 `lazyWaitClass`、固定 `expand: 400`；
- rAF 批处理（90-117 行）与 debounce（134-159 行）与主产物同源——实验变体不实验基础设施。

## 读完后想一件事

把这份文件与主产物 `lazysizes.js` 并排打开，逐个模块问「它为什么还需要存在」：答案是观察者模型吃掉了扫描、档位、节流三大件，留下的只有揭幕流程本身。**下一代架构的形态，往往早就写在你能读到的地方**——只差一个敢不敢换主产物的决策。
