# 用法面：index.html（官方演示页 = 这个库的 API 说明书）

这个页面本身就是最好的用法教程，每种标记形态配一段可运行的实例。注意两点：示例图片全是 2015 年代的外链（staticflickr 部分还活着，lorempixel 已停服），失效属正常——看机制（Network 面板、元素类的切换）不看图；页面在 134-146 行用 `lazySizesConfig` 全局变量调过参数（loadMode=1 演示保守模式、expand 按屏幕加大），不是默认值。

## 四种标记形态（按页面出现顺序）

1. **LQIP（低清占位图）**：`src="低清小图.jpg" data-src="高清图.jpg"`。低清图几 KB 随 HTML 就到，先撑住版面；替换成高清图时配 CSS 过渡（页面里的 `.blur-up` 思路：先模糊后清晰）；
2. **纯懒加载**：只有 `data-src`，没有 src。最省流量，缺点是 HTML 校验器会警告 img 无 src；
3. **响应式**：`data-srcset`（多分辨率候选）+ `data-sizes`（显示宽度提示）。全部 data- 前缀，就是为了**在加载前骗过浏览器的预加载器**（prefetcher 会扫 HTML 里的 src/srcset 提前下载，懒加载的第一步就是把这两个属性改名）；
4. **picture 元素/iframe**：picture 的 source 也用 data-srcset；iframe 用 data-src（第三方嵌入又重又慢，是懒加载的大户）。

## 两个进阶玩法

- **`data-expand` 负值做进场特效**（352 行起）：给容器设 `data-expand="-110"`，把「预载环」反向来收窄——元素要真正滚进视口一段才揭幕，配 CSS 的 `lazyload → lazyloaded` 过渡做出缩放淡入。**同一机制的正负两个方向**：正值提前加载买体验，负值推迟揭幕买特效；
- **动态插入的元素自动被发现**（664 行按钮）：点击往 DOM 里插新的懒加载图，不用调任何 API——MutationObserver 加活集合在起作用（见 lazysizes.js 解读第三节）。

## 值得注意的工程细节

- 页面加载了 `plugins/unveilhooks/ls.unveilhooks.min.js`（148 行）——插件把懒加载扩展到 script、背景图、视频，这是插件生态的入口，mature 案例展开；
- 老浏览器分支（9-16 行）：不支持 picture 元素时动态补 polyfill 脚本——那个时代「用新标准」等于「自己带补丁」。
