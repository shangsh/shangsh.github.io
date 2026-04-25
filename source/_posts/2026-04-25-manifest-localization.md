---
title: "网页应用清单的本地化支持

## 概述

网页应用清单（Web App Manifest）规范支持本地化，使开发者能够为不同语言和地区提供定制化的清单属性。这通过使用特殊格式的值来实现，允许根据用户的语言偏好加载相应的翻译。

## 实现机制

### 属性本地化

在清单文件中，可以使用带有语言标签后缀的属性名称来指定特定语言的变体。例如：

```json
{
  "name": "应用默认名称",
  "name_en-US": "Application Name",
  "name_zh-CN": "应用名称",
  "short_name": "默认",
  "short_name_en-US": "App",
  "short_name_zh-CN": "应用"
}
```

### 语言后缀格式

语言后缀使用下划线（_）连接语言代码和可选的区域代码。常见的格式包括：

- `name_en` - 英文
- `name_en-US` - 美国英文
- `name_zh-CN` - 简体中文（中国）
- `name_zh-TW` - 繁体中文（台湾）
- `name_fr-FR` - 法语（法国）

### 匹配算法

浏览器会根据以下优先级匹配最合适的本地化属性：

1. 精确匹配用户语言和区域设置
2. 仅匹配语言代码
3. 回退到不带语言后缀的默认属性

## 适用属性

支持本地化的清单属性包括：

- `name` / `short_name` - 应用名称
- `description` - 应用描述
- `launch_handler.client_mode` - 启动处理器的客户端模式
- `share_target.*` - 分享目标的文本字段

## 最佳实践

### 提供回退值

始终提供不带语言后缀的默认值，以确保在找不到匹配语言时应用仍能正常运行。

### 保持一致性

确保所有受支持语言的版本都包含相同的属性，保持翻译的完整性。

### 测试多语言场景

在不同语言设置下测试应用，确保本地化内容正确显示。

## 浏览器兼容性

主流浏览器均已支持网页应用清单的本地化功能，包括Chrome、Firefox、Safari和Edge。建议在生产环境中部署前进行充分测试。

## 示例

```json
{
  "name": "默认应用",
  "name_en": "Default App",
  "name_zh-CN": "默认应用",
  "name_ja": "デフォルトアプリ",
  "short_name": "应用",
  "short_name_en": "App",
  "short_name_zh-CN": "应用",
  "short_name_ja": "アプリ",
  "description": "一款优秀的应用",
  "description_en": "An excellent application",
  "description_zh-CN": "一款优秀的应用",
  "description_ja": "素晴らしいアプリケーション"
}
```

通过合理运用本地化支持，可以为全球用户提供更加本地化的网页应用安装体验。"
date: 2026-04-25 08:00:00
categories:
  - Chromium Blog
tags:
  - Chromium
  - Chrome
  - 翻译
---

> 原文：https://developer.chrome.com/blog/manifest-localization?hl=en
> 本文章由 AI 自动翻译自 Chromium 官方博客

您的清单文件现在支持多种语言。

---
*原文发布于 2026-04-25，由 haoge's Terminal 自动同步。*
