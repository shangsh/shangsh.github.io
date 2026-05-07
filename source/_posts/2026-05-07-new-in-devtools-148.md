---
title: "# DevTools (Chrome 148) 新变化

## 快捷方式 > 命令菜单中新增快捷键建议

现在，当你打开命令菜单（Command Menu）时，DevTools 会根据你的使用习惯，在命令列表顶部显示个性化的快捷键建议。快捷键建议根据你最近执行的操作动态调整。要访问命令菜单，请按 `Ctrl+Shift+P`（Windows/Linux）或 `Cmd+Shift+P`（macOS）。

![快捷键建议截图](https://developer.chrome.com/docs/devtools/command-menu/commandmenu-keyboard-shortcuts.png)

## 元素面板中的内联标签改进

现在，元素面板（Elements panel）的内联标签（inline标签）默认显示完整路径，不再截断。点击内联标签仍可打开相关菜单。你可以在设置（Settings）中切换回之前的折叠显示方式：取消勾选 **元素 > 在 DOM 树中显示完整路径**。

![内联标签改进截图](https://developer.chrome.com/docs/devtools/dom/inline-path.png)

## 样式面板中的改进

### 颜色和渐变现在支持复制 CSS

你现在可以在样式面板中复制任何颜色或渐变值。右键点击颜色或渐变旁边的指示器，然后选择 **复制**。之前只支持复制 `var()` 引用。

![颜色复制功能截图](https://developer.chrome.com/docs/devtools/css/reference.png)

### CSS 颜色语法高亮修复

之前，当 CSS 颜色值使用不正确的语法时，DevTools 会停止解析后续的颜色。现在已修复此问题，即使颜色语法有误，也能正确显示其他颜色的语法高亮。

## 网络面板中的改进

### 瀑布图中的资源优先级标记

网络面板（Network panel）的瀑布图（waterfall）现在显示了资源优先级标记，让你更清楚地了解资源加载顺序及其优化机会。

![优先级标记截图](https://developer.chrome.com/docs/devtools/network/reference.png)

## 性能面板中的新指标

### 总阻塞时间（TBT）指标

性能面板现在默认显示 **总阻塞时间（Total Blocking Time，TBT）** 指标。TBT 衡量的是主线程被阻塞足够长的时间以防止输入响应的总时间。你可以在 **性能指标**（Performance Insights）面板中找到此指标。

## 控制台中的改进

### 支持 `warn` 和 `error` 日志方法的自动补全

控制台（Console）现在为 `console.warn()` 和 `console.error()` 方法提供自动补全建议。输入 `warn` 或 `error` 后，DevTools 会自动建议对应的完整方法。

## 来源面板中的改进

### 在断点编辑器中启用/禁用断点

你可以通过点击断点旁边的复选框来启用或禁用断点，而无需打开或关闭所有断点。

![断点复选框截图](https://developer.chrome.com/docs/devtools/javascript/breakpoints.png)

## 开发者体验改进

### 更清晰的控制台消息

控制台消息现在使用了更清晰的措辞。例如，"Failed to load resource: the server responded with a status of 404" 改为 "404 (Not Found)"，使其更易阅读。

### 元素面包屑导航改进

元素面包屑现在支持更长的面包屑路径。当路径过长时，你可以在面包屑末尾看到省略号，点击可展开完整路径。

## 实验性功能

### 网络限制配置文件预设

你现在可以在网络限制（Network throttling）配置文件中添加自定义预设。自定义预设会显示在预设列表底部，并带有用户图标标记。

### 帧渲染器中的性能分析

现在你可以在性能面板中分析帧渲染器的性能。要启用此功能，请勾选 **设置 > 实验 > 性能 > 在性能面板中启用帧渲染器分析**。

## 相关资源

- 查看完整的 [Chrome DevTools 文档](https://developer.chrome.com/docs/devtools)
- 订阅 [Chrome DevTools 开发者频道](https://developers.google.com/webmasters/chrome-devtools/)
- 参与 [DevTools 社区讨论](https://groups.google.com/forum/#!forum/google-chrome-developer-tools)"
date: 2026-05-07 08:00:00
categories:
  - Chromium Blog
tags:
  - Chromium
  - Chrome
  - 翻译
---

> 原文：https://developer.chrome.com/blog/new-in-devtools-148?hl=en
> 本文章由 AI 自动翻译自 Chromium 官方博客

默认启用完整页面辅助功能树、广告来源工具提示、增强型推测规则调试，以及智能体 DevTools 重大更新

---
*原文发布于 2026-05-07，由 haoge's Terminal 自动同步。*
