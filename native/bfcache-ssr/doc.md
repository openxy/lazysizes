# SSR 与 bfcache 修补

## 本版本引入的概念

**兼容性长尾**：机制早已定形、平台已让位，维护期修的都是环境边角——但每个边角背后都是一类真实的工程场景：

- **SSR（服务端渲染）安全**：`typeof window != 'undefined' ? window : {}`（`lazysizes.js` 头部）——模块被 Node 引入（Next.js 等服务端渲染框架会全量 import）时不炸，到浏览器端再真正执行。「能在服务端被 import」本身成了库的兼容面；
- **Safari 后退缓存（bfcache）**：用户「后退」时 Safari 恢复整个页面快照，懒加载状态机却停在离开前——已加载的图被误判回未加载。修复靠 `elem.complete && elem.naturalWidth > 1` 这类「问实际状态而不是问记忆」的判定；
- **AMD 模式端正**：插件头的模块注册补全 AMD 分支——UMD 三分支（CommonJS/AMD/全局）的完整形态。

## 本版本的改动（差异面板对照上一版）

上游 5.0.0 → 5.2.2（5.1.x 并入此步）：主文件约 98+28 行变动，集中在头部模块注册、SSR 的 Date 注入（`factory(window, window.document, Date)`——SSR 环境下 Date 类可能丢，显式传进来）、可见性判断修正；插件侧 video-embed 补 YouTube 封面、blur-up 复制 style 属性。

这步的上游差异原有 60+ 文件：约 27 个 `.min.js` 是随源码改动的机械重压缩，二十多个插件 `.js` 是同一个 SSR 安全模式挨个铺开（概念就一行：`typeof window != 'undefined' ? window : 0`）。浏览视图已按 `.eduignore` 裁掉这些（被裁文件仍在运行树里），差异面板只剩概念载体。

## 思考题

- 为什么「被服务端 import 不炸」对一个浏览器端库变成了必修课？（提示：2019 年后 Next.js/Nuxt 类框架的普及方式）
- bfcache 修复为什么用 `naturalWidth` 而不是查类名状态机？（状态记忆 vs 实际状态的取舍）
- 维护期版本的 diff 特征是什么？（新增少、修补散、测试与文档占比高——对照 mature 案例第一版那种 445 行的机制 diff）
