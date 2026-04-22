---
title: "# 连接白名单源试用：保护 Web 应用程序的网络安全

## 概述

连接白名单（Connection Allowlists）是一项新的源试用功能，允许您控制页面可以连接到的网络端点。通过指定允许的连接目标，您可以防止敏感数据被发送到未经授权的服务器，并保护用户免受数据外泄和追踪的侵害。

## 工作原理

当您为页面启用连接白名单时，浏览器将拦截所有出站连接请求，并仅允许与预定义白名单规则匹配的连接。这些规则可以基于：

- **目标主机名**：指定允许连接的具体域名或通配符模式
- **目标端口**：限制允许连接的端口号
- **协议方案**：控制允许使用的协议（如 HTTPS、WSS）

## 配置方法

### 声明式声明

在 HTML 文档的 `<head>` 部分添加 `policy` 声明：

```html
<meta http-equiv="Permissions-Policy" content="allowlist=(destinations 'self' https://trusted-site.example.com)">
```

### HTTP 响应头

通过 `Permissions-Policy` 响应头配置：

```
Permissions-Policy: allowlist=(destinations 'self' https://trusted-site.example.com https://api.example.org)
```

## 安全优势

### 防止数据外泄

通过限制出站连接，您可以确保敏感信息（如用户凭证、个人数据）只能发送到经过验证的服务器端点。

### 抵御恶意软件通信

即使攻击者成功注入恶意代码，连接白名单也能阻止恶意软件与命令控制服务器的通信。

### 减少追踪风险

限制第三方连接可以有效防止跨站追踪和指纹识别。

## 最佳实践

1. **遵循最小权限原则**：仅允许应用程序正常运行所必需的连接
2. **定期审核白名单**：移除不再使用的端点，保持规则最新
3. **使用 HTTPS**：优先允许加密连接，保护数据传输安全
4. **测试覆盖**：确保所有合法的连接路径都经过测试

## 浏览器兼容性

连接白名单功能目前处于源试用阶段，支持 Chromium 内核浏览器。启用源试用令牌后，用户可以在受支持的浏览器中体验该功能。

## 参与源试用

如需参与源试用，请访问 Chrome 源试用页面注册并获取试用令牌。将获取的令牌添加到您的网页或服务器配置中，即可开始测试。

## 总结

连接白名单为 Web 应用程序提供了细粒度的网络连接控制能力。通过实施严格的连接策略，开发者可以显著增强应用程序的安全性，保护用户数据免受未授权访问和恶意利用。"
date: 2026-04-22 08:00:00
categories:
  - Chromium Blog
tags:
  - Chromium
  - Chrome
  - 翻译
---

> 原文：https://developer.chrome.com/blog/connection-allowlists-origin-trial?hl=en
> 本文章由 AI 自动翻译自 Chromium 官方博客

加入 Connection Allowlists 源试用，这是一项 Chrome 中的安全机制，可为文档和 Web Workers 创建网络沙箱。

---
*原文发布于 2026-04-22，由 haoge's Terminal 自动同步。*
