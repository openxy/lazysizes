# 成熟期核心：调度机制全家桶

## 本版本引入的概念

4.x 是这个库的成熟期：架构没变（还是 v1 系列那套事件轮询），但**调度机制全面升级**，插件生态成型。本版精读两件事：转正的调度全家桶，和一段没转正的实验。

**requestIdleCallback 转正**（`lazysizes.js:166-214`）：v1 系列里整段注释的实验版 throttle，在这里是正式实现——检查元素的活搬进浏览器空闲时段（idle 回调），彻底避开滚动渲染的关键路径。

**rAF 双队列批处理**（`lazysizes.js:112-149`）：所有 rAF 排队的活进一个批次，执行中新排的活进另一队列——帧内任务不无限膨胀，还能用 `_lsFlush` 强制清空。

**防抖收编**（`lazysizes.js:217-241`）：v1 注释里的防抖版 onload 也转正了——loadMode 档位切换从「定时器手搓」换成 debounce。

**一个没转正的实验**：`src/lazysizes-intersection.js`——用 IntersectionObserver **完整重写**了 loader：观察替代扫描、rootMargin 替代预载环、双观察器分「就绪」和「预载」两档。它就躺在这个版本的源码树里，从未构建成主产物。为什么？见下。

## 为什么 IO 版没转正（本案例最重要的工程决策之一)

- **硬依赖、无回退**：`src/lazysizes-intersection.js:13` 开头一行 `if(!window.IntersectionObserver || !MutationObserver){return;}`——不支持就整库静默失效。而 lazysizes 的立身之本是「零依赖跑遍所有浏览器」；
- **时间窗错位**：IO 在 4.x 时代（2016-2017）只有 Chrome/Firefox 刚跟上，Safari 要到 12.1（2019 年）才有。主产物换 IO 意味着放弃苹果用户；
- **等得起，也不用急**：轮询版虽是全量扫描，但节流+空闲调度打磨后性能够用——成熟库的哲学是「兼容面就是产品的一部分」，新能力先进实验通道养着。

结局反转：等 IO 普及了，平台直接给了 `loading="lazy"`（v5 系列），这个实验变体永远停在了 src 目录里。**下一代技术出现 ≠ 立刻换**，迁移决策要看用户面——这是读真实项目才能学到的判断。

## 怎么学这个版本

1. 先读 `doc/lazysizes.js.md`（与 v1 版同文件两代对照的读法，看「注释转正」的三处）；
2. 再读 `doc/src/lazysizes-intersection.js.md`（IO 实验变体——同一功能的另一种架构，两份解读对照读收获最大）；
3. 浏览 `plugins/` 目录感受生态形态（unveilhooks/bgset/rias/object-fit 等二十来个，各自解决一类加载问题）。

## 思考题

- 对比两版 loader：IO 版删掉了哪些代码（scroll 监听、checkElements 循环、loadMode 状态机……）？删掉的每一块，对应 IO 的哪个能力？
- IO 版为什么还需要 MutationObserver？（提示：新插入 DOM 的元素谁来登记？）
- 如果你是维护者，2019 年 Safari 支持 IO 后你会把主产物切过去吗？考虑一下已装旧版的存量站点。
