---
title: "Nuxt 静态生成踩坑"
date: 2022-02-02T13:42:26+08:00
isCJKlanguage: true
categories:
  - '前端框架'
tags:
  - 'Nuxt'
  - 'Vue 2'
  - 'SSG'
description: '前段时间将一个 Go 与 Vue 2 的项目改成用 Nuxt 全站静态生成，踩了一些坑，在这儿总结一下'
ShowToc: false
draft: false
---

## URL 中含中文带来的问题

Nuxt 2.15.8 有个问题：若 URL 中含非 ASCII 字符，即使请求是在`fetch`和`asyncData`里发起的，生成后的页面仍会请求 API。

原因是生成过程中，未能正确转义非 ASCII 字符，导致生成后无法访问对应页面，回退到单页应用，执行请求 API 逻辑。

该问题已在 Pull Request 9494[^non-ascii-decode]中修复。修复已计划合并到 2.16.0 中。但截至本文编写时，最新 release 还是 2.15.8，所以要在自己的应用中修复问题，需要卸载原本的`nuxt`包，安装`nuxt-edge`包；或者可以关注 2.16.0 的发布状态[^nuxt-2.16.0-pr]。

[^non-ascii-decode]: [fix(vue-renderer): decode route path for `payload.js` #9494](https://github.com/nuxt/nuxt.js/pull/9494)

[^nuxt-2.16.0-pr]: [v2.16.0 #9660](https://github.com/nuxt/nuxt.js/pull/9660)

## `http_proxy`环境变量拖慢生成过程

我不确定这是不是 Nuxt 框架的问题，因为我在 Nuxt 仓库中没有搜到相关报告。不过在我的机器上，只要使用了`http_proxy`环境变量，整个生成过程就会变得非常缓慢，暂时清空`http_proxy`后问题消失。

## 其他

接下来是我认为使用 Nuxt 框架要注意的一些地方。

### asyncData、fetch 和 Vue 的生命周期钩子

建议在使用 Nuxt 框架前，要对[框架生命周期](https://nuxtjs.org/docs/concepts/nuxt-lifecycle)做到心中有数。

据我所知，在`asyncData`和`fetch`之外的钩子里请求 API，生成之后的页面也会请求，所以要注意请求时机。

### 告诉框架要生成哪些路径

Nuxt 2.13 之后，生成过程会使用爬虫，爬虫会跟着`<NuxtLink>`里的路径生成页面。如果应用中有些路径无法从`<NuxtLink>`访问，则需要告诉框架需要生成哪些路径，具体如何操作可以看[官方文档](https://nuxtjs.org/docs/configuration-glossary/configuration-generate#routes)。

### 生成页面时请求 API 的频率

太快会把服务器打挂，然后生成过程就会卡住，框架也不报错。可以看[官方文档](https://nuxtjs.org/docs/configuration-glossary/configuration-generate#interval)设置请求间隔。
