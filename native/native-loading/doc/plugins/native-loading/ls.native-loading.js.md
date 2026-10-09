# 精读：plugins/native-loading/ls.native-loading.js（让位协议——库把舞台交给平台）

140 行的过渡插件。目标只有一个：**浏览器支持原生 `loading` 属性时，让库闭嘴让路**。它不是加功能，是拆自己。按「加载 → 侦测 → 拆除 → 让位」四段读。

## 一、插件的挂载方式（1-22 行）

UMD 三分支：CommonJS 用 `require('lazysizes')`；全局已有时直接装；**主库还没加载完成时，先监听 `lazyunveilread` 事件等它出生再装**（7-9 行的 globalInstall）。插件与宿主的双向解耦，值得抄。

## 二、能力侦测（24-26、39-41 行）

```js
var imgSupport = 'loading' in HTMLImageElement.prototype;
var iframeSupport = 'loading' in HTMLIFrameElement.prototype;
```

特性检测（feature detection）标准写法——探测能力而不是嗅探浏览器版本。不支持 loading 也没有 MutationObserver 就整插件退出（39 行）：老浏览器继续用库的完整机制，**插件只在新浏览器上生效**。

## 三、拆除自己的监听（43-72 行）

`disableEvents()` 把宿主库挂在 window/document 上的 scroll/resize/focus/mouseover/click 等监听**全部移除**——正是 polling 案例里读到的那张宽事件面。对照着看很戏剧：一个里程碑前它们是性能调优的对象，现在整个成了不需要存在的东西（原生属性在浏览器内部处理，JS 层无需任何监听）。71 行 `removeALSL` 还留了个 1 秒延时的保险——拆监听这种事也要防时序。

## 四、让位逻辑（74-103 行）

- `runConfig()`：首次用到时才配置（惰性初始化）；
- `setLoadingAttribute` 打开时，在 `lazybeforeunveil` 事件里给元素补 `loading="lazy"` 属性（95-101 行）——**库的揭幕事件流保持不变，只是揭幕动作从「换 src」变成「打原生标记」**，下游插件无感；
- `prematureUnveil`（105-121 行）是精髓：宿主在扫描中发现元素带原生能力时，这个钩子返回 true 告诉宿主「这张图交出去，你别管」——返回 true 的意思不是「加载完了」，是「**不归你加载**」。旧钩子链式保留（119-120 行），多个拦截层叠不互踩。

## 五、读完对照

- 库的完整机制（polling 案例读的那 700 行）在这里被压缩成「谁问了才答一句：归平台」；
- 这个插件的形态就是**库退役前留给用户的楼梯**：标记语言不用改（还是 data-src），浏览器新就享受原生，浏览器老就走老路——渐进增强（progressive enhancement）从口号落成了代码；
- 工程判断的样本：作者没有把主库改成「原生优先」，而是让过渡以插件形态存在——**主库的稳定性承诺不因平台演进而破产**，想吃新能力的人自己装插件。
