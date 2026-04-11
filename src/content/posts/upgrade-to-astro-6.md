---
title: 从 Astro 5 升级到 Astro 6 的记录
published: 2026-04-11
description: 这次把 Fuwari 从 Astro 5.13.10 升到 Astro 6.1.5 时，实际遇到的问题和修复步骤。
tags: [Astro, Migration, Upgrade]
category: Guides
draft: false
lang: zh_CN
---

# 背景

这次升级的直接原因，是项目准备继续引入新的官方集成，而 `@astrojs/vue@6` 已经要求 `astro@^6.0.0`。项目原本使用的是 `Astro 5.13.10`，所以不能只装新集成而不升级主框架。

升级前先确认两件事：

1. 本地和部署环境都要切到 `Node 22.12.0+`
2. 不要只改 `package.json`，先阅读 Astro 官方升级指南

官方推荐的升级命令是：

```bash
pnpm dlx @astrojs/upgrade
```

## 这次升级里最容易踩坑的地方

### 1. `src/content/config.ts` 不能再直接沿用

Astro 6 要求把内容集合配置放到：

```text
src/content.config.ts
```

只把文件移动过去还不够。升级后如果集合没有配置 `loader`，构建阶段会直接把集合视为空集合，然后出现这类报错：

```text
The collection "posts" does not exist or is empty.
About page content not found
```

修复方式是给每个 collection 显式声明 `glob()` loader：

```ts
import { defineCollection } from "astro:content";
import { glob } from "astro/loaders";
import { z } from "astro/zod";

const postsCollection = defineCollection({
  loader: glob({
    pattern: "**/*.{md,mdx}",
    base: "./src/content/posts",
  }),
  schema: z.object({
    title: z.string(),
    published: z.coerce.date(),
  }),
});
```

### 2. `entry.slug` 需要改成 `entry.id`

升级到 Astro 6 之后，内容条目已经不再暴露旧的 `slug` 字段。之前像下面这种写法都会报错：

```ts
params: { slug: entry.slug }
link: `/posts/${post.slug}/`
```

统一改成：

```ts
params: { slug: entry.id }
link: `/posts/${post.id}/`
```

### 3. `entry.render()` 需要改成 `render(entry)`

旧代码里如果直接这样写：

```ts
const { Content } = await entry.render();
```

升级后会看到类似报错：

```text
Property 'render' does not exist on type ...
```

正确写法是从 `astro:content` 导入 `render`：

```ts
import { render } from "astro:content";

const { Content, headings, remarkPluginFrontmatter } = await render(entry);
```

### 4. 嵌套文章的相对图片路径要注意

这个项目里有类似 `src/content/posts/guide/index.md` 这样的嵌套文章，并且封面图片是相对路径：

```yaml
image: "./cover.jpeg"
```

在 Astro 5 的旧逻辑里，代码通过 `entry.id` 还能拿到接近文件路径的值；升级到 Astro 6 后，`entry.id` 变成了面向 URL 的 slug，再继续拿它推导目录，就可能把图片目录算错。

更稳妥的做法是改用 `entry.filePath` 推导文章所在目录，再去解析相对图片。

## 实际修改清单

这次迁移里做了这些改动：

- 把内容配置迁到 `src/content.config.ts`
- 为 `posts` 和 `spec` 补上 `glob()` loader
- 把 schema 里的日期改成 `z.coerce.date()`
- 把所有 `entry.slug` 替换为 `entry.id`
- 把所有 `entry.render()` 替换为 `render(entry)`
- 用 `entry.filePath` 修正嵌套文章封面图的相对路径解析

## 升级后建议执行的检查

每次迁移完成后，我会至少跑下面两条命令：

```bash
pnpm check
pnpm build
```

如果 `check` 通过但 `build` 失败，不要只盯着 TypeScript 报错。像内容集合为空、某篇文章取不到、相对图片路径错误，这类问题更容易在真正构建静态页面时暴露出来。

## 总结

这次从 Astro 5 升到 Astro 6，真正需要处理的重点其实不是依赖版本号本身，而是 **content collections 的新规则** 和 **旧内容条目 API 的替换**。

只要把这两块迁完整，再补一轮构建验证，升级难度是可控的。真正浪费时间的，通常都是“已经改了一半，所以看起来像是能跑，但在构建阶段才报空集合”这种半迁移状态。
