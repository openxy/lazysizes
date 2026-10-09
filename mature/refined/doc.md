# 成熟期收尾与 IO 实验变体

## 本版本引入的概念

成熟期的收尾版：机制在前两版已定形，这版全是小数点战争式的修补——但**src/ 目录里躺着本案例最重要的一份代码**：IntersectionObserver 版完整重写。

**收尾修补**（4.1.0，核心文件仅 16 行变动）：`elem.complete && elem.naturalWidth > 1` 判真加载（躲缓存返回 0 尺寸的怪癖）、lowRuns 边界 off-by-one 修正、爬虫检测微调。读它的意义在于看清「成熟」= 不再长新东西。

**IO 实验变体**（`src/lazysizes-intersection.js`，540 行）：同一功能的另一种架构——

- 双观察器：`inviewObserver`（无 rootMargin，真进视口才动）+ `preloadObserver`（rootMargin 外扩 = 预载环的观察者化）；
- scroll 监听、checkElements 全量循环、loadMode 档位、throttle **全部消失**——浏览器只在你关心的事发生时叫你一次，拉（轮询）变推（通知）；
- MutationObserver 登记新元素，挂 observe 后立刻把 lazyload 类换成 lazyWaitClass——**用类的更替当登记簿**，活集合不再重复吐旧元素；
- 揭幕后 `unobserve`——观察成本只覆盖未加载元素，越用越轻。

**为什么从未转正**（本案例集最重要的工程决策样本）：头部一行 `if(!IO || !MutationObserver){return;}` 硬依赖无回退，而 4.x 时代（2016-2017）Safari 还没有 IO（要到 12.1，2019 年）；lazysizes 的立身之本是「零依赖跑遍所有浏览器」。结局反转：等 IO 普及了，平台直接给了 `loading="lazy"`（native 案例）——实验变体永远停在 src 目录里。**下一代技术出现 ≠ 立刻换**，迁移决策要看用户面。

## 怎么学这个版本

1. 差异面板对照上一版：体会 16 行的收尾 diff（成熟期的常态）；
2. 精读 `doc/src/lazysizes-intersection.js.md`（IO 变体逐段导读），与主产物并排打开，逐个模块问「它为什么还需要存在」；
3. `doc/lazysizes.js.md` 是 4.1.0 主文件的完整精读（调度机制全景——前两版的概念在这里的最终形态），可作本案例的总复习。

## 思考题

- IO 版为什么还需要 MutationObserver？（新插入的元素谁来登记？）
- 2019 年 Safari 支持 IO 后，你会把主产物切过去吗？考虑已装旧版的存量站点。
- 观察者架构吃掉了扫描、档位、节流三大件，但 `lazyQuedElements` 预载队列还在——底层模型的先进为什么消不掉上层的并发管理？
