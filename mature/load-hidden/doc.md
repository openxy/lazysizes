# loadHidden 与插件 CommonJS 化

## 本版本引入的概念

**loadHidden 开关**（4.0.0）：隐藏元素（`visibility: hidden`）默认**也加载**——因为很多站点用「先隐藏再淡入」的特效，不加载会穿帮。但这费流量，于是做成配置开关（`lazySizesConfig.loadHidden: false` 关掉）。**把一个有争议的行为变成选项**，是成熟库处理用户分歧的标准手法：两种用法都对，让用户选。

**插件 CommonJS 化**（4.0.0）：二十多个插件全部改造，能在 npm/webpack 环境里 `require` 使用——库正式进入前端工程化时代（2016 年 npm 革命）。

**rAF 双队列**（3.x 吸收）：批处理从单队列升为 `firstFns`/`secondFns` 双队列——执行中又排进来的活进另一队列，帧内任务不无限膨胀；后台标签页用 setTimeout 顶替停摆的 rAF；暴露 `_lsFlush` 强制清空口（初始化后立即检查首屏用）。

## 本版本的改动（差异面板对照上一版）

上游 2.0.0 → 4.0.1（中间的 3.x 已并入此步）：

- rAF 批处理双队列化（`lazysizes.js:112-149`）；
- checkElements 主循环加 loadHidden 判断（`lazysizes.js:394`）；
- 爬虫正则扩为 `/(gle|ing)bot/`（Googlebot 加 Bingbot：爬虫没视口，全部直接揭幕喂给它——SEO 饭碗）；
- 配置表新增 loadHidden/ricTimeout/throttleDelay；expFactor 1.7→1.5、expand 档位简化为两档；
- `plugins/` 目录从几个长到二十几个（bgset、rias、object-fit、artdirection、blur-up……）——差异面板里成片的新增文件就是生态本身，浏览目录比逐个读文件有用。

## 思考题

- 默认「隐藏元素也加载」是性能逆向的选择，作者为什么仍这么定默认值？（提示：特效站坏图是 bug，多花流量只是成本）
- 插件从「script 标签先后加载」到「CommonJS 依赖注入」，挂载机制改了什么？（对照 native 案例里 native-loading 插件的三分支头）
- 双队列批处理防的是什么病？（提示：回调里又 rAF 排队的递归）
