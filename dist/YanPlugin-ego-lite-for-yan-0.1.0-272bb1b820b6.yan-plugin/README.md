# YanPlugin-ego-lite-for-Yan

<img src="assets/logo.png" width="88" height="88" alt="ego (lite) for Yan" align="right">

> 给 [Yan Agent](https://github.com/) 的 ego lite 原生插件：普通网页任务走 ego 的独立 Space，本地预览与前端验收仍走 Yan 内置浏览器。

## 这是什么

[ego (lite)](https://github.com/citrolabs/ego-lite) 是一个 Chromium 浏览器，让人和 AI Agent 共用同一套登录态、但各占独立工作区（Space）。

上游已经为九种 Agent 宿主提供了插件包，但没有 Yan。直接拿 `.claude-plugin` / `.codex-plugin` 包走 Yan 的兼容导入也能用，只是会留下四个问题：

| 兼容导入的问题 | 本插件的处理 |
| --- | --- |
| 就绪检查不认 CLI，缺 `ego-browser` 也报 `ready: true` | 声明 `requiredCommands: ["ego-browser"]`，缺命令直接不可用 |
| 打包 40 个文件，含 `index.js`、`dsh/`、5 份宿主清单等死重量 | 只保留 4 个文件：清单、说明、Skill、文档 |
| Skill 原文要求「优先 ego，压过内置浏览器」 | 明确路由边界：本地预览与前端验收仍用内置浏览器 |
| 截图默认落在系统临时目录 | 约定写入 `.yanagent/evidence/` |

**不复制上游的自动化文档。** 上游的 `SKILL.md` 本身只是个薄入口，真正的内容由 `ego-browser skill` 按已安装版本动态输出。本插件沿用这个设计，所以 ego lite 升级后不需要同步 API 文档。

## 安装

本插件是原生 Yan 插件包（`yan-plugin.json`，schemaVersion 1）。

1. 从 Releases 或 `dist/` 取 `.yan-plugin` 包，或用本仓库根目录直接安装。
2. 通过 Yan 的插件安装入口安装并启用。
3. 首次启用后确认 `ego-browser` 可解析。

> 安装入口是否开放取决于你的 Yan 版本与策略，本仓库不提供绕过安装校验的脚本。

## 前置条件

- 已安装 ego lite（Windows beta / macOS），并完成首次引导
- `ego-browser` 在 PATH 上：

```bash
ego-browser --version
ego-browser nodejs -e "console.log('ego-browser ready')"
```

第二条输出 `ego-browser ready` 才算就绪。Windows 上如果安装后当前终端找不到命令，重开终端；仍找不到时检查 ego lite 是否完成了引导。

## 路由边界

| 场景 | 工具 |
| --- | --- |
| 检索、阅读、登录后台、填表、抓取、多页调研 | ego-browser |
| 本地 `file://` 页面预览 | Yan 内置浏览器 |
| 前端验收：视口切换、截图对比、交互确认 | Yan 内置浏览器 |

ego 强在登录态继承、独立 Space 和「一条脚本跑完多步」；内置浏览器强在可见、可控视口、可直接截图验收。两者互补，不互相替代。

## 使用

启用后，Agent 会先执行：

```bash
ego-browser skill
```

拿到版本匹配的指南，再写脚本。示例（一次调用完成导航、等待、抽取、截图）：

```bash
ego-browser nodejs <<'EOF'
const task = await taskSpace("Yan ego task");
const page = task.page("p1");
await page.goto("https://example.com");
await page.waitForSelector("body", { state: "visible", timeout: 15000 });
const shot = await page.screenshot({ path: "/abs/path/.yanagent/evidence/page.png" });
console.log(JSON.stringify({ spaceId: task.spaceId, title: await page.title(), shot }));
EOF
```

## 目录结构

```
yan-plugin.json                    # 清单：依赖、平台、Skill 声明
instructions.md                    # 插件级说明：路由边界、失败语义
skills/ego-browser/SKILL.md        # 薄入口 + Yan 约定
dist/                              # 校验后生成的 .yan-plugin 包
```

## 版本与同步

- 插件版本：`0.1.0`
- 适配的 ego-browser Skill 指南版本：`2.1.0`（ego lite `0.6.1.13` 实测）
- 上游 Skill 升级时，本仓库只需要核对 `skills/ego-browser/SKILL.md` 里的引导命令是否仍为 `ego-browser skill`，无需同步 API 文档。

## 验证状态

- 清单与资源：通过 `yan_skills_create_plugin` 校验
- 运行时：在 ego lite `0.6.1.13` + `ego-browser` `0.6.1.13` 上完成 readiness 检查与一次真实页面任务（Bing 搜索 + 结果抽取 + 截图）
- 未验证：macOS 环境；ego lite 未来版本的 CLI 变更

## 许可

MIT。浏览器自动化能力来自 [citrolabs/ego-lite](https://github.com/citrolabs/ego-lite)（MIT），归属见 `NOTICE`。
