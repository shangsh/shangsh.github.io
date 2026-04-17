---
title: "# Chrome 147 支持并发和嵌套的视图过渡，并引入元素作用域视图过渡

视图过渡 API 最初设计用于在单页应用（SPA）的两个文档状态之间创建流畅的视觉过渡效果。现在，Chrome 147 为这一 API 带来了重大升级，引入了对并发视图过渡、嵌套视图过渡以及元素作用域视图过渡的支持。

## 并发视图过渡

现在您可以在同一个页面中同时运行多个独立的视图过渡。每个视图过渡可以拥有自己的 `ViewTransition` 对象，允许不同的 UI 区域独立进行动画过渡，而不会相互干扰。

## 嵌套视图过渡

视图过渡现在支持嵌套结构。一个视图过渡可以在另一个视图过渡内部启动，形成层级关系。这种设计模式特别适合复杂的多步骤交互场景，例如表单填写流程或向导式界面。

## 元素作用域视图过渡

这一特性允许开发者将视图过渡的效果限定在特定 DOM 元素范围内，而不是整个文档。通过为特定元素定义过渡动画，可以实现更精细的控制和更好的性能表现。

```javascript
// 元素作用域视图过渡示例
element1.startViewTransition({
  type: 'nested',
  update: updateFunction,
  types: ['slide', 'fade']
}, element2);
```

## 浏览器兼容性

这些新特性已在 Chrome 147 中正式推出，为开发者提供了更强大的工具来创建流畅、连贯的用户界面体验。视图过渡 API 的持续演进使得实现应用内状态转换变得更加简单和直观。"
date: 2026-04-17 08:00:00
categories:
  - Chromium Blog
tags:
  - Chromium
  - Chrome
  - 翻译
---

> 原文：https://developer.chrome.com/blog/element-scoped-view-transitions?hl=en
> 本文章由 AI 自动翻译自 Chromium 官方博客

视图转换的下一个迭代版本已正式到来！

---
*原文发布于 2026-04-17，由 haoge's Terminal 自动同步。*
