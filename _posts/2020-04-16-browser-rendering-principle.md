---
layout: post
title: 浏览器的渲染：过程与原理
date: 2020-04-16
desc: "浏览器的渲染过程与渲染原理"
keywords: "browser, render"
categories: [Http]
tags: [Browser]
icon:
---

## 前言

本文不是关于浏览器渲染的底层原理或前端优化具体细节的讲解，而是关于浏览器对页面的渲染——这一过程的描述及其背后原理的解释。这是因为前端优化是一个非常庞大且零散的知识集合，一篇文章如果要写优化的具体方法恐怕只能做一些有限的列举。

然而，如果了解清楚浏览器的渲染过程、渲染原理，其实就掌握了指导原则。根据优化原则，可以实现出无数种具体的优化方案，各种预编译、预加载、资源合并、按需加载方案都是针对浏览器渲染过程的优化。

## 关键渲染路径

提到页面渲染，有几个相关度非常高的概念，最重要的是关键渲染路径，其他几个概念都可以从它展开，下面稍作说明。

**关键渲染路径（Critical Rendering Path）**是指与当前用户操作有关的内容。例如用户刚刚打开一个页面，首屏的显示就是当前用户操作相关的内容，具体就是浏览器收到 HTML、CSS 和 JavaScript 等资源并对其进行处理，从而渲染出 Web 页面。

了解浏览器渲染的过程与原理，很大程度上是为了**优化关键渲染路径**，但优化应该是针对具体问题的解决方案，所以优化没有一定之规。例如为了保障**首屏内容**的最快速显示，通常会提到**渐进式页面渲染**，但是为了渐进式页面渲染，就需要做资源的拆分，那么以什么粒度拆分、要不要拆分，不同页面、不同场景 策略不同。具体方案的确定既要考虑体验问题，也要考虑工程问题。

## 浏览器渲染页面的过程

从耗时的角度，浏览器请求、加载、渲染一个页面，时间花在下面[五件事情](https://developers.google.com/speed/docs/insights/mobile 'hp')上：

1. DNS 查询
2. TCP 连接
3. HTTP 请求及响应
4. 服务器响应
5. 客户端渲染

本文讨论第五部分，即浏览器对内容的渲染，这一部分（渲染树构建、布局及绘制），又可以分为下面[五个步骤](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-tree-construction 'hp')：

1. 处理 HTML 标记并构建 DOM 树；
2. 处理 CSS 标记并构建 CSSOM 树；
3. 将 DOM 与 CSSOM 合并成一个渲染树；
4. 根据渲染树来布局，以计算每个节点的几何信息；
5. 将各个节点绘制到屏幕上。

需要明白，这五个步骤并不一定一次性顺序完成。如果 DOM 或 CSSOM 被修改，以上过程需要重复执行，这样才能计算出哪些像素需要在屏幕上进行重新渲染。实际页面中 CSS 与 JavaScript 往往会多次修改 DOM 和 CSSOM，下面就来看看它们的影响方式。

## 阻塞渲染：CSS 与 JavaScript

谈论资源的阻塞时，我们要清楚，现代浏览器总是并行加载资源。例如，当 HTML 解析器（HTML Parser）被脚本阻塞时，解析器虽然会停止构建 DOM，但仍然会识别该脚本后面的资源，并进行预加载。

同时，由于下面两点：

1. 默认情况下，CSS 被视为阻塞渲染的资源，这意味着浏览器将不会渲染任何已处理的内容，直至 CSSOM 构建完毕；
2. JavaScript 不仅可以读取和修改 DOM 属性，还可以读取和修改 CSSOM 属性。

存在阻塞的 CSS 资源时，浏览器会延迟 JavaScript 的执行和 DOM 的构建。另外：

1. 当浏览器遇到一个 `script` 标记时，DOM 构建将暂停，直至脚本完成执行；
2. JavaScript 可以读取和修改 DOM 与 CSSOM；
3. CSSOM 构建时，JavaScript 执行将暂停，直至 CSSOM 就绪。

所以，`script` 标签的位置很重要。实际使用时，可以遵循下面两个原则：

1. CSS 优先：引入顺序上，CSS 资源优先于 JavaScript 资源；
2. JavaScript 应尽量少影响 DOM 的构建。

浏览器的发展日益加快，具体的渲染策略会不断进化，但了解这些原理后，就能想通它进化的逻辑。下面来看看 CSS 与 JavaScript 具体会怎样阻塞资源。

### CSS

```html
<style> p { color: red; }</style>
<link rel="stylesheet" href="index.css">
```

这样的 `link` 标签（无论是否 inline）会被视为阻塞渲染的资源，浏览器会优先处理这些 CSS 资源，直至 CSSOM 构建完毕。

渲染树（Render-Tree）的关键渲染路径中，要求同时具有 DOM 和 CSSOM，之后才会构建渲染树。即 HTML 和 CSS 都是阻塞渲染的资源。HTML 显然是必需的，因为包括我们希望显示的文本在内的内容，都在 DOM 中存放，那么可以从 CSS 上想办法。

最容易想到的当然是**精简 CSS 并尽快提供它**。除此之外，还可以用媒体类型（media type）和媒体查询（media query）来解除对渲染的阻塞。

```html
<link href="index.css" rel="stylesheet">
<link href="print.css" rel="stylesheet" media="print">
<link href="other.css" ref="stylesheet" media="(min-width: 30em) and (orientation: landscape)">
```

第一个资源会加载并阻塞。

第二个资源设置了媒体类型，会加载但不会阻塞，`print` 声明只在打印网页时使用。

第三个资源提供了媒体查询，会在符合条件时阻塞渲染。

### JavaScript





