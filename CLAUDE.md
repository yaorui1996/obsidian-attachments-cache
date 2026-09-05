# CLAUDE.md — obsidian-attachments-cache

Fork 自 luisbs/obsidian-attachments-cache（v0.7.0）。与上游的差异和开发上下文记录在这里，随仓库走。

## 本 fork 做了什么

### LP（Live Preview）自动缓存功能（2026-09-05）

**问题**：原插件只靠 `registerMarkdownPostProcessor` 自动缓存，但 PostProcessor 只在阅读模式触发；LP/Source 模式正文由 CodeMirror widget 渲染，不走渲染管线，所以 LP 下重启/切文档/滑动都不会缓存（实测确认）。

**思路**：编辑流不依赖渲染管线，直接对**编辑器文本**做扫描——复用 `detectRemotes()`（src/commons/EditorFunctions.ts）从纯文本提取外链，逐个调 `api.cache()`。

**改动**（5 个文件）：
- `src/commons/PluginSettings.ts`：+`handle_onedit` 设置（默认 true）
- `src/settings/i18n/en.ts` + `types.d.ts`：+`handleOneditName` 文案
- `src/settings/PluginSettingTab.ts`：Triggers 区 +1 toggle
- `src/main.ts`：`active-leaf-change`（打开/切文档，立即扫描）+ `editor-change`（防抖 1s）→ `#handleEditorContent()`；`onunload` 清理 `window.clearTimeout(this.#editDebouncer)`

**关键设计决策**：
1. 编辑流**永远只 cache 不改写文本**（`api.cache()` 而非 `api.archive()`）——即使 CacheRule 开了 archive 也不在编辑流里改，正在编辑时改文本会打断光标。archive 文本改写仍由阅读模式 PostProcessor 和手动命令负责。
2. URL 用 Set 去重后串行下载，防同一 URL 并发竞态。
3. 防重复下载不新写机制，靠 API 已有的 memo Map（`AttachmentsCacheApi.#memo`）+ `vault.getFileByPath` 存在性检查，重复扫描代价极低。

## 关键认知（避免重新踩坑）

- **缓存判断链没有扩展名白名单**：http 开头 → 域名过 CacheRule remotes 黑白名单（默认 `w *` 全放行）→ URL/frontmatter 覆盖参数 → `URI.hasExt` 仅检查"URL 末端带点"（不验证扩展名真伪）。`.zip`/`.php`/`.my123` 都会被下载。README 的 "Supported attachments file-formats" 列表**不对应代码过滤逻辑**，是上游文档缺陷（实际只影响"缓存后能否渲染显示"）。
- main.ts 唯一的扩展名正则（图片列表）只决定缓存后的显示方式（替换 img.src vs `![[local]]` 重渲染），不参与"是否缓存"判断。
- 无扩展名 URL（动态图床）缓存不了，是架构硬限制；TODO.md 里的 `{type}` 扩展组是计划中未做的功能。
- tsc 有 4 个**上游预存**的报错（`PluginState.test.ts` 缺 archive 字段），与新改动无关。

## 开发与验证

```bash
pnpm watch      # dev 构建，直接输出 test-vault/.obsidian/plugins/attachments-cache/（带 sourcemap），配合 test-vault 的 hot-reload 插件
pnpm build:dist # 正式构建 → dist/（无 sourcemap）
npx vitest run -c ./scripts/vite.config.dist.mjs   # 测试（11 个）
```

- `test-vault/example/` 有测试素材：10 张图（png/jpeg/jpg/webp/avif/gif/bmp/svg/ico/tiff）+ document.pdf
- 测 LP 功能：LP 模式打开含外链的笔记 → 看缓存目录生成文件；手输新链接停 1s → 触发缓存

## 可能的下一步

- 扩展名白/黑名单功能（收窄缓存范围，避免网页链接也被下载）
- 无扩展名 URL 支持（按响应 Content-Type 补扩展名）

## 改动记录

### 2026-09-05 LP 自动缓存（handle_onedit）

首个 fork 功能。背景、思路与设计决策见上文「本 fork 做了什么」；实测素材在 `test-vault/example/`（10 种格式图片 + PDF）。

配套文档：**README 结构重排**——原上游 README 保留为 `README.upstream.md`（git mv，历史连续），新 README.md 只含 fork 声明（起点=上游仓库）+ Fork features 功能介绍，**中英双语**。README 风格（用户多轮纠正后定型）：**面向使用者，不写实现细节**（不提事件名/防抖/函数名），只讲"什么场景下获得什么功能 + 配置入口 + 限制"；**不用表格**，按功能小节组织，标题带日期。**约定：以后每个重要功能提交都同步——README.md 中英各加一个功能小节（场景+功能+配置，不写实现）；CLAUDE.md 改动记录加一条（实现细节记这里）。**
