---
name: ego-browser
description: 用 ego lite 执行网页任务：打开站点、搜索、读取页面、填表、点击、截图与批量抽取。仅在需要真实浏览器、登录态或并行 Space 时使用；本地 file:// 预览与前端验收改用 Yan 内置浏览器。
---

# ego-browser（Yan 适配版）

开始任何浏览器任务前先运行：

```bash
ego-browser skill
```

并严格按它的输出执行。它打印与已安装 ego lite 版本匹配的使用指南；升级 ego lite 之后重新运行一次。

如果输出末尾没有 `[ego-browser:skill] end of guide`，说明内容被截断：先写到文件，再分段读取。

```bash
ego-browser skill > /tmp/ego-browser-skill.md
```

## Yan 环境约定

1. **路由**：普通网页任务（检索、阅读、登录后台、填表、抓取）用 ego；本地 `file://` 预览、视口切换、截图验收用 Yan 内置浏览器。
2. **一个目标一个 Space**：`taskSpace("<简短名称>")`，把 `spaceId` 打印出来；后续轮次按 id 恢复，不要为了「恢复失败」另开新 Space。
3. **证据**：截图写到当前工作区的 `.yanagent/evidence/`，用正斜杠绝对路径。不要留在系统临时目录。
4. **不碰用户空间**：`openedBy: "unknown"` 视为用户所有，不关闭；不关闭用户自己创建的标签页。
5. **需要人处理时停下**：登录、验证码、付款、最终提交都由用户确认，不要代替用户点击。
6. **失败语义**：`ego-browser` 报未找到时，直接说明「ego lite 未安装或 CLI 未注册到 PATH」，不要自行修改 PATH、注册表或重装。

## 执行习惯

- 每个脚本调用结束前打印下一步需要的状态（`spaceId`、`page.label`、快照或提取结果）。
- `page.url()`、`page.title()`、`snapshot()` 等都是异步的，必须 `await`，否则会得到 `{}`。
- 页面变化后旧快照引用失效，重新取快照再用新的 ref。
- 结果已经满足要求就停止，不要用多种方式重复确认同一件事。

## 参考

- `ego-browser skill` 的完整指南：版本匹配，优先于任何静态文档。
- 本插件 `instructions.md`：路由边界与失败处理。
