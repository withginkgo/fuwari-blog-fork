---
title: 在当前 Astro 6 项目里接入 Vue，并修掉按钮发黑问题
published: 2026-03-12
description: 记录这次给 Astro 6 博客接入 Vue 的过程，以及随后修复 categories、配色切换和明暗切换按钮发黑问题的思路。
tags: [Astro, Vue, Debugging, UI]
category: Frontend
draft: false
lang: zh_CN
---

# 这次改了什么

这次没有改博客本身的整体结构，而是在现有的 `Astro + Svelte` 项目里补上了 `Vue` 支持，并加了一个独立 demo 页面。

新增的入口是：

```text
/vue-demo/
```

这页的作用很直接：

- 验证 `@astrojs/vue` 已经正常接入
- 验证 `.vue` 组件可以在当前项目里作为 island 使用
- 不动原有 Svelte 组件，保持现有结构继续可用

## Vue 接入做法

这次接入保持了最小改动：

- 安装 `@astrojs/vue` 和 `vue`
- 在 `astro.config.mjs` 里注册 `vue()`
- 新增一个 Vue 组件作为 demo
- 新增一个 Astro 页面专门承载这个 demo

也就是说，这个仓库现在不是“从 Svelte 改成 Vue”，而是“在 Astro 项目里继续保留原有 Svelte，并额外支持 Vue”。

## Demo 页面做了什么

`/vue-demo/` 里放了一个简单的计数器组件，用来验证三件事：

1. `.vue` 单文件组件可以被 Astro 正常导入
2. 组件可以通过 `client:load` 正常 hydration
3. Vue 的本地状态更新在当前项目里工作正常

如果后面要继续在这个项目里加 Vue 交互组件，可以直接沿用这个模式。

## 后面为什么又开始修按钮样式

Vue 接入之后，项目本身能跑，但在实际点页面时暴露出一串 UI 问题：

- `Categories` 列表项目会莫名出现黑色填充
- 配色切换按钮会出现不对的底色
- 明暗切换面板里的按钮悬浮后会出现蓝色四角和黑线

这些问题看起来像是同一种现象，但根因其实有两层。

## 第一层问题：无效的按钮结构

`Categories` 列表原本用的是：

```html
<a>
  <button>...</button>
</a>
```

这是无效 HTML。浏览器在这种结构上会掺进自己的默认按钮绘制，结果就会出现和设计稿不一致的背景、边框或者焦点态。

所以这里的修法不是继续补样式，而是把结构改成单层可点击元素：

```html
<a>...</a>
```

这一步修掉了 categories 列表里那种最明显的黑色填充问题。

## 第二层问题：把 `bg-none` 当成了透明背景

后面明暗切换和配色切换还会出问题，根因更底层。

项目里不少按钮类用了 `bg-none`，但它实际表示的是“没有背景图片”，并不等于“透明背景”。  
对于原生 `<button>`，浏览器自己的默认背景、边框和 focus 外观仍然可能存在。

所以后面做了几件事：

- 把按钮相关样式里的 `bg-none` 改成真正的 `bg-transparent`
- 给按钮类补 `appearance: none`、`border: 0`、`outline: none`、`box-shadow: none`
- 给实际的原生按钮补 `type="button"`

## 为什么刷新后一开始正常，过一会又出问题

这个现象说明不只是“浏览器默认按钮样式”这么简单。

后面继续排查后发现，`btn-plain scale-animation` 这套样式本来是靠 `::before` 做悬浮背景动画的。它放在链接上没问题，但放在原生 `button` 上，某些浏览器环境里会和按钮自身绘制叠在一起，最后表现成蓝色角、黑线或者奇怪的填充。

最后的处理方式是：

- 保留链接元素上的 `scale-animation` 效果
- 对原生 `button.btn-plain.scale-animation` 单独做覆盖
- 不再让这些按钮依赖 `::before` 动画层
- 改成直接使用普通的 `hover:bg-*` / `active:bg-*`

这一步之后，明暗切换和配色切换按钮才真正稳定下来。

## 这次改动的边界

这轮处理的原则一直没变：

- 不改博客整体结构
- 不把原有 Svelte 组件整体迁到 Vue
- 只增加一个 Vue demo 页面
- 只修复已经暴露出来的按钮和交互样式问题

## 当前结果

现在这个仓库已经具备下面几个状态：

- Astro 6 正常工作
- Vue 已经接入
- `/vue-demo/` 可访问
- categories 列表不再出现异常黑色填充
- 配色切换和明暗切换按钮不再出现那种浏览器默认按钮外观污染

这次我也重新跑过：

```bash
pnpm check
pnpm build
```

都通过了。

## 如果后面继续扩展

后面如果要继续在这个项目里用 Vue，比较稳妥的做法是：

- 新增交互组件优先考虑统一用 Vue
- 原有 Svelte 先不动
- 按 island 的方式渐进式增加，而不是一次性重写

这样可以把演进成本压到最低，也不会把当前可用的博客结构打散。
