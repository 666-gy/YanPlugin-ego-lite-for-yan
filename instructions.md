# ego (lite) for Yan

把 ego lite 作为 Yan Agent 的网页执行环境：普通网页任务在 ego 的独立 Space 里跑，你自己的标签页不受影响。

## 前置条件

1. 已安装 ego lite（Windows beta 或 macOS），并至少启动过一次完成引导。
2. `ego-browser` 命令在 PATH 上可解析。验证：

   ```bash
   ego-browser --version
   ego-browser nodejs -e "console.log('ego-browser ready')"
   ```

   第二条输出 `ego-browser ready` 才算就绪。

本插件只声明依赖，不会自动安装 ego lite，也不会修改 PATH。

## 路由边界

| 场景 | 用哪个 |
| --- | --- |
| 检索、阅读、登录后台、填表、抓取、多页调研 | ego-browser |
| 本地 `file://` 页面预览 | Yan 内置浏览器 |
| 前端验收：视口切换、截图对比、交互确认 | Yan 内置浏览器 |

ego 的优势是「你的登录态 + 独立 Space + 一条脚本跑完多步」；内置浏览器的优势是「在 Yan 面板里可见、可控视口、可直接截图验收」。两者不互相替代。

## 工作方式

- 每次任务先执行 `ego-browser skill`，拿与已安装版本匹配的指南，再写脚本。
- 一个目标用一个 Space：`taskSpace("简短名称")`，记住 `spaceId`，后续轮次按 id 恢复。
- 截图写到当前工作区的 `.yanagent/evidence/`，使用正斜杠绝对路径。
- 遇到登录、验证码、付款或提交类操作，停下来交给用户。

## 失败处理

`ego-browser` 报未找到时，说明「ego lite 未安装或 CLI 未注册到 PATH」，不要自行改 PATH、改注册表或重装。ego 未运行时提示用户启动，再重试一次；仍失败则回退 Yan 内置浏览器并说明原因。

## 与上游的关系

浏览器自动化能力全部来自 [citrolabs/ego-lite](https://github.com/citrolabs/ego-lite)（MIT）。本插件只提供 Yan 侧的薄适配：清单、路由边界、证据约定与失败语义，不复制上游的自动化 API 文档——那些内容由 `ego-browser skill` 按版本动态提供。
