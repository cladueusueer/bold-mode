# bold-mode

**风险可以留给想法，不能留给波及范围。**

一个 Claude Code 插件，让 Claude 敢于尝试大胆的想法：重写、删掉一层、假设"这块如果不存在会怎样"，同时不让正在工作的代码冒险。每次实验都跑在隔离副本里，受硬性步数预算约束，最后给出一份对照基线的如实成绩单。

[![tests](https://github.com/cladueusueer/bold-mode/actions/workflows/test.yml/badge.svg)](https://github.com/cladueusueer/bold-mode/actions/workflows/test.yml)
[![license: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Claude Code plugin](https://img.shields.io/badge/Claude%20Code-plugin-ffb020.svg)](#install)
[![good first issues](https://img.shields.io/github/issues/cladueusueer/bold-mode/good%20first%20issue?label=good%20first%20issues&color=5fd38d)](https://github.com/cladueusueer/bold-mode/labels/good%20first%20issue)

[English](README.md) · [עברית](README.he.md)

![An illustrative bold-mode run: isolate, baseline, build, measure, report](assets/demo.svg)

<sub>报告格式示意，非真实项目的测量结果。完整示例见 [examples/sample-report.md](examples/sample-report.md)。</sub>

## 为什么需要它

让 AI 改进一段代码，通常得到的是稳妥、渐进的方案。这是合理的默认，但有时真正的收获来自那个不太舒服的想法，尝试它要花时间，也有搞乱的风险。

bold-mode 让大胆尝试变得**低成本且安全**：

| 没有 bold-mode | 有 bold-mode |
|---|---|
| 稳妥、渐进的建议 | 稳妥、大胆甚至激进的想法，都构建并测量 |
| "看起来更干净" | 一张对照基线的数字表 |
| 在工作区里大段重写 | 只在 git worktree 或 `/tmp` 副本里改动 |
| 实验停不下来 | 由 hook 强制执行硬性步数预算 |
| 大胆想法"总是赢" | 如实报告，包括"大胆方案落败" |

## 包含什么

- **`bold-mode` skill**：方法本身。隔离、基线、分出稳妥/大胆/激进三类思路，构建、测量、报告、移交。
- **11 个透镜**：简短的重构思路手册（删一层、反转流程、数据优先于代码、十倍放大等），供 Claude 生成不落俗套的想法。[浏览透镜列表](skills/bold-mode/lenses/README.md)。
- **守卫 hook**：仅在大胆运行期间生效。它会：
  - 统计步数，预算用完即**停止运行**（默认 25 步），
  - **阻止改动原始文件**（改动必须发生在隔离副本），
  - **阻止 force-push、推送到 main/master/release、发布、部署、基础设施变更和删除数据**，
  - 在原始目录内阻止 `git reset --hard`、`git clean`、`git merge` 和递归删除。
- **`/bold` 命令**：一行启动一次运行。

## 安装

在 Claude Code 中：

```
/plugin marketplace add cladueusueer/bold-mode
/plugin install bold-mode@bold-mode
```

重启 Claude Code（或新开会话）让 hook 加载。守卫依赖 PATH 中的 `python3`；没有它，skill 仍可用，但不会执行强制拦截。

**仅 skill**（不带 hook，适用于 Claude Code、claude.ai 及其他读取 `SKILL.md` 的 agent）：把 `skills/bold-mode/` 复制到 `~/.claude/skills/bold-mode/`，或将该文件夹打包上传为 claude.ai 的 skill。

## 用法

```
/bold the caching layer in src/cache — it's 900 lines and still slow
```

也可以直接这样说：

```
Be bold with the onboarding flow. Use bold-mode.
```

会得到类似下面这样的结果（示意）：

| | 基线 | 大胆：删掉缓存层，改用 HTTP 缓存 | 稳妥：调 TTL | 激进：构建期预计算 |
|---|---|---|---|---|
| 检查通过 | 42/42 | 42/42 | 未构建（预估 42/42） | 39/42 |
| 行数 | 912 | 140 | 约 900 | 210 |
| p95 延迟 | 180 ms | 95 ms | 约 160 ms | 20 ms |
| 采用风险 | – | 中：取决于 CDN 配置 | 低 | 高：数据过期 |
| 采用工作量 | – | 半天 | 一小时 | 两天 |

**建议**：采用大胆方案，结果在分支 `bold/cache`（worktree `../app-bold-cache`）。

完整示例见 [examples/sample-report.md](examples/sample-report.md)。

## 什么时候不要用

小修小补、bug 修复、错别字、常规重构，以及任何已经稳定的代码。skill 对此有明确说明，且只在被明确要求时触发。

## 设置

| 变量 | 默认值 | 作用 |
|---|---|---|
| `BOLD_MODE_BUDGET` | `25` | 每次运行计入的最大步数（Bash / Edit / Write）；读取不计步数。 |
| `BOLD_MODE_GUARD` | `on` | 设为 `off` 关闭所有拦截（步数预算与波及范围保护）。 |

## 守卫如何判断一次运行是否处于激活状态

当调用 `bold-mode` skill，或提示以 `/bold` 开头（或提到 `bold-mode`）时开始一次运行；Claude 结束本轮即结束。状态按会话存在临时目录。运行之外，hook 不做任何事。

## 如实说明的局限

- 守卫看到的是工具调用，不是意图。用重定向（`>`）写入原始文件的 Bash 命令，或通过脚本写入的操作，不会被捕获。主要保护来自 skill 指引和隔离副本，hook 是安全带。
- 命令模式是黑名单，不是沙箱。需要硬性保证时，请在容器里运行 Claude Code。
- "测量"结果取决于测试本身。没有测试时，skill 会写几条快速检查，并说明对比结论偏弱。

## 参与贡献

新人可以从 [good first issue](https://github.com/cladueusueer/bold-mode/labels/good%20first%20issue) 入手，多数是"写一个透镜"（一个简短 markdown 文件）。

最简单的贡献是**新增一个透镜**：一个简短 markdown 文件，记录一个帮你找到更好方案的重构思路。见 [CONTRIBUTING.md](CONTRIBUTING.md)。欢迎提交守卫规则改进、翻译和真实项目报告。

## 许可证

MIT
