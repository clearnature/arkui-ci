# arkui-ci — CI 治理仓（arkui-dom-runtime 的治理与代码分离）

本仓承载 [arkui-dom-runtime](https://github.com/clearnature/arkui-dom-runtime) 的
**全部 CI 编排逻辑**（reusable workflows）；代码仓的 `.github/workflows/` 只保留
**薄委托层**（≤15 行：触发面 + `uses:` 指向本仓 + SHA 钉死）。

## 治理语义（流量保护）

| 层 | 触发 | 内容 |
|---|---|---|
| **pr-gate 前置门** | PR 打开/推送（免审自动）| PR body 三要素声明检查 + 快检（分钟级）|
| **重矩阵（审核制）** | 维护者加 `ci/full` 标签 / push main·ci / 手动 | 六引擎全量（15-60 分钟×腿）|
| **打包** | push main / 手动 | Electron 目录分发 tar.gz + 包内冒烟 |

原则：免审 PR 只烧前置门的分钟数；重算力必须有人工审核动作（加标签）才启动。
分析风险、决定方向 → 加 `ci/full` → 全量跑，这就是「审核制」的落地动作。

## Reusable Workflows（代码仓薄委托的指向）

| 文件 | 用途 |
|---|---|
| `nodejs-toolchain.yml` | V8 工具链快检（build-runtime 一致性 + 抽取冒烟）|
| `nodejs-browser.yml` | Chrome 浏览器全矩阵（虚拟时间，含豁免清单机制）|
| `electron-realtime.yml` | Electron 44 真实时钟 97 用例（时序权威端）|
| `firefox-matrix.yml` | Firefox(Gecko) 87 用例（经典 WebDriver）|
| `webkit-matrix.yml` | WebKitGTK 87 用例（playwright）|
| `ohos-clt.yml` | HarmonyOS CLT 编译 + AOT + fixtures 漂移守门 |
| `kernel-contract.yml` | 内核运行时契约（冻结 .so 直载）|
| `kernel-compilers.yml` | 五语言编译器（源码 → .so → 契约 5/5）|
| `pr-gate.yml` | PR 前置门（body 声明检查 + 快检）|

## 变更纪律

- 改本仓 = 改治理：走本仓 PR 审核，合入后由**代码仓 bump `@SHA`** 的一行 PR 生效；
- 代码仓的薄委托全部 SHA 钉死（供应链纪律，同指纹步）；
- 自检：`selftest.yml` 在本仓 push 时跑轻量 reusable（证明定义可解析可执行）。
