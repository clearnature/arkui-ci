# arkui-ci — 通用 CI 治理仓（任何项目可用的 reusable workflow 库）

本仓是**与具体项目解耦**的 CI 治理仓：reusable workflows 只定义【编排骨架 +
runner 环境 + 治理语义（两级门/留档/并发）】，项目差异全部通过 `inputs`
（setup-command / test-command / artifact-path）注入——**任何项目把自己的
测试入口命令传进来即可复用**，本仓不含任何项目专属路径。

## 治理语义（GOVERNANCE.md 为唯一权威）

| 层 | 触发 | 内容 | 审核要求 |
|---|---|---|---|
| pr-gate 前置门 | PR 打开/推送（免审自动） | 声明检查 + 快检 | 无 |
| 重矩阵全量 | push main/ci / `ci/full` 标签（审核制）/ 手动 | 各引擎全量 | 加标签 = 风险分析与方向裁定完成 |

## Reusable Workflows

| 文件 | 引擎 | 关键 inputs |
|---|---|---|
| `browser.yml` | Chrome headless | setup-command / test-command |
| `firefox.yml` | Firefox 经典 WebDriver | 同上 |
| `webkit.yml` | WebKitGTK（playwright，版本可钉） | 同上 + playwright-version |
| `electron.yml` | Electron 真实时钟（xvfb + metacity） | 同上 + electron-version |
| `kernel-contract.yml` | 内核运行时契约（冻结 .so 直载） | node-version |
| `kernel-compilers.yml` | 五语言编译器（gcc/go/rust/ghc/cjc） | —（自包含） |
| `pr-gate.yml` | PR 前置门（body 声明检查 + 快检） | pr-number / pr-body |

## 使用（项目侧薄委托示例）

```yaml
jobs:
  browser:
    uses: <本仓>/.github/workflows/browser.yml@<SHA 钉死>
    with:
      test-command: bash run.sh all
      setup-command: sudo apt-get install -y fonts-noto-cjk
```

## 变更纪律

- 改本仓 = 改治理：走本仓 PR 审核；
- 使用方仓库通过 **bump `@SHA`** 的一行 PR 生效（治理变更显式化）。
