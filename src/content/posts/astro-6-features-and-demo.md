---
title: Astro 6 新特性梳理和当前项目 Demo
published: 2026-03-11
description: 基于当前已经迁到 Astro 6 的博客项目，梳理 Astro 6 的主要新特性，并落一个可访问 demo。
tags: [Astro, Demo, Migration]
category: Frontend
draft: false
lang: zh_CN
---

# Astro 6 带来了什么

Astro 官方在 `2026-03-10` 发布的 `Astro 6.0` 里，重点提到的能力包括：

- 内置 Fonts API
- 稳定版 Live Content Collections
- 稳定版 Content Security Policy API
- 重构后的 `astro dev`
- 围绕 Node 22 / Vite 7 / Zod 4 的底层升级
- 实验性的 Rust compiler、queued rendering、route caching

官方发布说明：

- [Astro 6.0 release post](https://astro.build/blog/astro-6/)
- [Upgrade to Astro v6](https://docs.astro.build/en/guides/upgrade-to/v6/)
- [Fonts guide](https://docs.astro.build/en/guides/fonts/)

## 哪些适合在这个项目里直接试

不是每个新特性都适合这个仓库马上落地。

这个项目是一个静态博客模板，所以我优先挑了两类：

1. **能直接看见效果的**
2. **和当前迁移结果本身直接相关的**

最后选的是：

- `Fonts API`
- `Content Layer API` 在当前项目里的实际使用结果

## 已做的 Demo

这次加了一个独立页面：

```text
/astro-6-demo/
```

这个页面演示了两件事：

### 1. Astro 6 Fonts API

在 `astro.config.mjs` 里为 `Roboto` 配置了 Astro 6 的 `fonts` 选项，并通过 `local()` provider 指向仓库里的本地字体文件。然后在 demo 页面里通过 `<Font />` 把字体注入到页面头部，并把主标题切到这套字体变量上。

这意味着这个页面不是继续依赖旧的全局 `@fontsource` CSS 导入方式，而是直接走 Astro 6 提供的字体能力。

### 2. 当前项目迁好的 Content Layer

页面会读取当前 `posts` collection，展示最新文章列表。这个列表直接依赖迁移后的：

- `src/content.config.ts`
- `glob()` loader
- Astro 6 的 collection entry API

所以它不只是一个展示页，也顺带验证了这次内容层迁移已经在实际页面里正常工作。

在线查看：

- [Astro 6 Demo](/astro-6-demo/)

## 为什么没有把所有新特性都硬塞进来

有些 Astro 6 新特性虽然很强，但不适合这个仓库直接塞一个“看起来像 demo”的实现。

比如：

- `Live Content Collections` 更适合有外部 CMS 或 API 的场景
- `CSP API` 更偏安全配置和部署策略，不是最适合当前模板直接做成页面演示的能力
- `Queued Rendering`、`Rust compiler`、`Route Caching` 目前仍偏实验性

所以这次 demo 的思路不是“功能越多越好”，而是“在当前项目里做一个真实可用、可访问、可解释的 Astro 6 示例”。

## 访问方式

本地启动项目后，可以直接访问：

```text
/astro-6-demo/
```

如果你希望，我下一步还可以继续做第二轮 Astro 6 demo，例如：

- 把整个站点的主字体切到 Fonts API
- 尝试给当前项目启用 CSP API
- 单独做一个针对 Live Content Collections 的示例路由
