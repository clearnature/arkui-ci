# CI 治理规则（GOVERNANCE）

> 本文件是治理语义的唯一权威；workflow 文件是实现，语义以本文件为准。
> 变更本文件 = 治理变更，走本仓 PR 审核。

## 1. 两级门（流量保护）

| 级别 | 触发 | 覆盖 | 审核要求 |
|---|---|---|---|
| **pr-gate 前置门** | PR 打开/推送（免审自动） | body 三要素声明检查 + 快检 | 无 |
| **重矩阵全量** | ① push main/ci ② `ci/full` 标签（审核制）③ 手动 | 六引擎全量 + AOT + 漂移守门 | 加标签 = 维护者已完成风险分析与方向裁定 |

## 2. PR body 三要素（pr-gate 强制）

1. **变更说明**——改了什么；
2. **风险与影响面**——可能破坏什么；
3. **验证**——怎么证明没破坏（本地命令 + 结果）。

缺任一：pr-gate 红，PR 不得进入重矩阵（加 `ci/full` 也不行——先补声明）。

## 3. GitHub Actions Schema 约束（R159.3 十八跑实证）

### 3a. reusable caller job 不允许 `env:`

`jobs.<id>.env` 在 `uses:` 类型的 job 里**不被 GitHub schema 允许**——即使 YAML 语法合法，
GitHub 在 startup 阶段即拒绝（0 job、无日志、UI 只给一句 "workflow file issue"）。
需要传环境变量给被调用方时：**内联到 `with.test-command` 前缀**（如
`ARKUI_XXX=yyy bash run.sh all`）或改用 `secrets: inherit`。

### 3b. reusable workflow 顶层 `concurrency` 触发 startup failure

被调用 workflow（`workflow_call` 触发）的顶层 `concurrency:` 块会导致
GitHub startup 阶段拒绝（0 job、0 日志——十七跑实证：有 concurrency 三条全挂、
无 concurrency 三条全绿）。**并发控制放在调用方 workflow 顶层**，
reusable 内不加。

### 3c. reusable workflow 文件必须在 `.github/workflows/`

放在仓库根目录或其他位置的 `.yml` **不会被 GitHub 注册为 workflow**——
即使用 `uses:` 引用也 404。治理仓的 reusable 一律在
`.github/workflows/` 下。

### 3d. 包库 ABI tag 跨发行不同

ghcup/bindist 安装的 GHC 包库文件名含发行特有的 ABI tag
（本地 bindist=`-inplace-`，CI ghcup=`-e31e-`）。**不可按固定文件名 dlopen**——
用 dirent 前缀扫描（contract_common.c `try_hs_lib2`）。

## 4. 虚拟时间豁免清单（builtindemo / imageext / measimage）

依据（十四跑实证）：真图像解码的完成投递与动画收口在
`--virtual-time-budget` + 负载下会无限迟/时序形状失真。这三页的权威 =
**真实时钟端**（Electron 腿 / firefox / webkit / android / 本地）。
浏览器虚拟时间腿经 `ARKUI_VIRTUAL_TIME_SKIP` 跳过并留痕。
机理记档：代码仓 `runtime/ohos-shims.js` 解码注释。

## 5. 仓颉运行时不再分发

仓颉运行时（31MB 华为 nightly 二进制）不入任何仓：CI 的 cjk 系用例 SKIP
留痕（同 kernel-contract 的 SKIP 机制）。cjk 权威 = 本地（CANGJIE_RT_LIB
指向 nightly）+ 鸿蒙模拟器端 + 打包态。

## 6. runner 钉版

`ubuntu-24.04`（2026-10-02 全矩阵实测校准）。ubuntu-latest 将于
2026-10-19 迁 Ubuntu 26——26 的适配（包名/预装变化）作为独立验证切片，
验证绿之前不随 latest 漂移。

Windows 腿（`browser-windows.yml`）用 `windows-latest`：Chrome 依赖
runner 镜像预装（`C:\Program Files\Google\Chrome\Application\chrome.exe`），
预装路径跨镜像版本稳定，钉版收益低。**计费 2×**（Windows runner 分钟
系数 2），只承接确需 Windows 覆盖的腿。**shell 纪律**：job 级
`defaults: { run: { shell: bash } }`，所有 run 步骤走 Git Bash——
项目注入的 setup/test 命令同样按 bash 语义书写，严禁依赖 PowerShell。

## 7. 标签语义

| 标签 | 语义 |
|---|---|
| `ci/full` | 审核通过：对本 PR 触发六条重矩阵全量 |
