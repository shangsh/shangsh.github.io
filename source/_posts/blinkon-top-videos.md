---
title: BlinkOn Talks 观看量最高的3个视频 - 技术干货整理
date: 2026-03-22 16:40:00
tags:
  - BlinkOn
  - Chrome
  - Chromium
  - V8
categories:
  - 技术解读
---

# BlinkOn Talks 观看量最高的视频整理

BlinkOn 是 Google Chrome 团队的内部技术会议，分享浏览器内核、渲染引擎、V8 等核心技术。本文整理了 @BlinkOn Talks YouTube 频道观看量最高的 3 个视频。

---

# 1. Life of a Pixel (29,001 次观看)

**视频地址**: https://www.youtube.com/watch?v=K2QHdgAKP-s

## 内容简介

"Life of a Pixel" 是 Chrome 团队新人入职的必学资料，讲解 Chromium 如何从 HTML/CSS/JS 渲染到屏幕上的像素。

## 渲染流程

演讲跟踪了 Web 内容到显示像素的所有步骤：

1. **HTML → DOM Tree** - 解析 HTML 构建 DOM 树
2. **CSS → Style** - 整合 CSS 构建 Render Tree
3. **Layout** - 排版到 Paint Tree
4. **Paint → Layer** - 转到 Layer Tree
5. **Compositing** - 合成线程分块
6. **Rasterization** - 光栅化到 GPU
7. **Display** - 最终显示到屏幕

## 重要特性

- LayoutNG - 全新布局引擎
- Vulkan - GPU 渲染升级
- Slim Paint - 精简绘制流程

---

# 2. BlinkOn 6: Ignition - V8 Interpreter (22,203 次观看)

**视频地址**: https://www.youtube.com/watch?v=r5OWCtuKiAk

## 内容简介

介绍 V8 引擎的解释器 Ignition，负责快速启动和字节码生成。

## 核心概念

- **Ignition**: V8 的解释器，用于快速启动
- **TurboFan**: 优化编译器，基于类型反馈进行优化
- **JIT**: 即时编译策略

## 浏览器引擎

- WebKit → Blink: Google 从 WebKit 分支创建 Blink
- V8: Chrome 的 JavaScript 引擎，采用 JIT 编译

---

# 3. QUIC 101 (21,614 次观看)

**视频地址**: https://www.youtube.com/watch?v=dQ5AND4DPyU

## QUIC 协议简介

QUIC (Quick UDP Internet Connections) 是 Google 开发的传输层协议，后演进为 HTTP/3 标准。

## 核心优势

1. **减少连接延迟** - 0-RTT/1-RTT 连接建立
2. **改善丢包恢复** - UDP + 独立流控制
3. **连接迁移** - 支持网络切换
4. **消除队头阻塞** - 多路复用独立流

## 在 Chrome 中的应用

- 通过 `chrome://flags` 可启用实验性 QUIC
- 服务器返回 Alt-Svc 响应头表示支持 HTTP/3

---

# 其他热门视频

| 观看量 | 标题 |
|--------|------|
| 15,252 | Chromium Overview |
| 9,114 | Chrome University: Life of a Script |
| 8,880 | Life of a Pixel (Chrome University 2020) |
| 8,004 | Headless Chrome |
| 7,115 | Block Layout Deep Dive |
| 7,049 | Life of a Navigation |

---

*来源: YouTube @BlinkOn Talks 频道*
