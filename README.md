# lazysizes · 读一个真实前端项目的演进

案例集 = 上游真实项目 [aFarkas/lazysizes](https://github.com/aFarkas/lazysizes)（MIT 许可，图片/iframe 懒加载库）按教学选版的镜像：每个版本挂一个上游 tag（`from:` 坐标，构建期物化原样代码），教学侧只提供解读（doc.md 与文件级解读），不改上游一个字节。

## 案例组织：按里程碑拆案例，案例内多版本成链

真实项目提交极多，按里程碑拆成三个案例（每个案例 = 一组相关功能 = 一个里程碑），**案例内版本成链、相邻版本可差异对照**，案例边界放在架构切换处（跨边界的 diff 噪声太大，不适合教学）：

| 案例 | 里程碑 | 版本链（上游 tag） |
|---|---|---|
| polling | 手写轮询实现（1.x，2015） | core(1.3.1) → svg-layout(1.4.0) → umd-split(1.5.0) |
| mature | 调度成熟与生态（2.x–4.x，2016-17） | idle-scheduling(2.0.0) → load-hidden(4.0.1) → refined(4.1.0) |
| native | 平台收编（5.x，2018-19） | native-loading(5.0.0) → bfcache-ssr(5.2.2) → final(5.3.1，末 tag) |

整个案例集讲一条完整的技术普及弧线：**手写实现 → 库繁荣 → 平台原生 → 库退役**。注意一个反直觉的事实：IntersectionObserver 上市时这个库做过完整重写（`src/lazysizes-intersection.js`，mature/refined 版本树里可读到），但主产物始终没换——兼容包袱与用户面怎么影响技术决策，是本案例集最值得学的判断题。

## 怎么学

1. 进案例 → 沿版本链走，先读版本文档的「本版本的改动」，再开差异面板对照上一版（每个案例三版、两段 diff，体量都在可精读范围）；
2. 文件级解读（doc/）给关键文件的逐段导读：主文件 `lazysizes.js` 的完整精读在 polling/umd-split，IO 实验变体与让位协议的精读在 mature/refined 与 native/native-loading；
3. 案例之间的衔接靠各版本文档与上表叙述（跨里程碑不 diff，读叙事）；
4. 代码可在线编辑试跑（编辑只保存在本机浏览器，上游代码树只读——受约束是特性）；
5. 演示页 `index.html` 的示例图是 2015 年代外链，部分已失效——看加载机制（Network 面板、类切换）不看图。
