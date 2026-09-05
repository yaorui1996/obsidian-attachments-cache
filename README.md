# Attachments Cache (fork)

**English** | [中文](#中文)

Fork of [luisbs/obsidian-attachments-cache](https://github.com/luisbs/obsidian-attachments-cache) — an Obsidian plugin that stores remote attachments locally inside the vault.

- Upstream documentation: [README.upstream.md](./README.upstream.md) · [Settings docs](./docs/settings.md) · [Upstream changelog](./CHANGELOG.md)

## Fork features

### Live Preview / Source mode caching (2026-09-05)

The upstream plugin only auto-caches attachments in **Reading View**. This fork brings automatic caching to **Live Preview and Source mode**: when you open a note containing remote attachments, they are downloaded to the vault automatically; when you add new remote links while editing, they get cached as well.

As with upstream, the note text is never modified — only downloaded. The trigger can be toggled in **Settings → Triggers** ([docs](./docs/settings.md)).

> Like upstream, caching is not restricted to a fixed list of file types: any URL that passes the CacheRule remotes check will be cached. Narrow the remotes list if you only want images.

---

## 中文

**Fork 自 [luisbs/obsidian-attachments-cache](https://github.com/luisbs/obsidian-attachments-cache)** —— 一个把远程附件缓存到 vault 本地的 Obsidian 插件。

- 上游文档：[README.upstream.md](./README.upstream.md) · [设置文档](./docs/settings.md) · [上游更新日志](./CHANGELOG.md)

## Fork 新增功能

### 实时预览/源码模式缓存（2026-09-05）

上游插件只在**阅读模式**下自动缓存附件。本 fork 把自动缓存带到了**实时预览和源码模式**：打开包含远程附件的笔记时，附件会自动下载到 vault；编辑中新增的远程链接也会被自动缓存。

与上游一致，此功能**只下载，不修改笔记文本**。可在 **设置 → Triggers**（[文档](./docs/settings.md)）中开关。

> 与上游相同，缓存不限定文件类型：凡是通过 CacheRule remotes 检查的 URL 都会被缓存。若只想缓存图片，请收窄 remotes 列表。
