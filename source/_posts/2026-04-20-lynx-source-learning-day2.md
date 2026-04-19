---
title: "Lynx 源码学习 Day 2：ScreenSize、Frame 与 FakeView"
date: 2026-04-20 00:00:00
tags: [Lynx, C++, Chromium, 源码解析, DuiLib]
categories: 技术分享
---

> 这是 Day 1 的后续，建议先看 [Day 1：从 main 到屏幕像素](/2026-04-20-lynx-source-learning-day1/)

---

## 二、ScreenSize vs Frame — 窗口和视口的区别

这是 Day 1 的疑问点，搞清楚后很简单。

### 场景：1920×1080 的显示器上开了个 800×600 的窗口

```
┌─────────────────────────────────────┐
│  屏幕 1920×1080 │ ← ScreenSize 描述这个
│  │
│  ┌──────────────┐  │
│  │ 窗口 800×600 │  │ ← Frame 描述这个
│  │  │  │
│  │ Lynx在这里画 │  │
│  └──────────────┘  │
└─────────────────────────────────────┘
```

### ScreenSize = 屏幕的物理信息

```cpp
SetScreenSize(1920, 1080, 1.5)  // 屏幕宽高 + DPI缩放
```

告诉 JS 引擎："你跑在一个 1.5 倍缩放的屏幕上。"

**为什么需要？** JS 里写 `font-size: 14px`，这个 14px 到底多大？得看 DPI。1.5 倍屏上 14px = 21 物理像素。

类比 DuiLib：`GetSystemMetrics(SM_CXSCREEN)` + DPI 感知。

### Frame = Lynx 渲染区域

```cpp
SetFrame(0, 0, 800, 600)  // x, y, 宽, 高
```

告诉渲染引擎："你在窗口里这个区域画。"

类比 DuiLib：`MoveWindow(hwnd, 0, 0, 800, 600)`。

### 实际例子：左边原生列表 + 右边 Lynx 聊天区

```
┌──────────┬──────────────────────┐
│ 联系人列表 │      聊天区域        │
│ (Win32)  │      (Lynx)         │
│ 原生控件  │ Frame(200,0,600,800) │
└──────────┴──────────────────────┘
```

ScreenSize 还是全屏信息，Frame 只占右半边。

**大多数情况下 ScreenSize 和 Frame 宽高一样**（Lynx 占满窗口），只有嵌入部分区域时才不同。

|        | ScreenSize | Frame                  |
|--------|------------|------------------------|
| 描述什么 | 屏幕的物理信息 | Lynx 渲染区域          |
| DuiLib 类比 | `GetSystemMetrics()` + DPI | `MoveWindow()` |
| 给谁用  | JS 引擎算像素 | 渲染引擎画图           |

---

## 三、FakeView — 自定义原生控件

### 从 DuiLib 说起

DuiLib 里在 XML 嵌一个 ActiveX 控件：

```xml
<ActiveX name="webview" clsid="{8856F961-340A-11D0-A96B-00C04FD705A2}"/>
```

XML 是占位符，真正干活的是 C++ 创建的 COM 对象。

### Lynx 里一模一样

JS 里写：

```jsx
<x-fake-view style="width: 100%; height: 200px;" />
```

这个 `<x-fake-view>` 只是占位符，**真正创建的是 C++ 的 `FakeView` 对象**。

```cpp
// 注册：JS 遇到 <x-fake-view> 就创建 FakeView
builder.RegisterNativeView<FakeView>("x-fake-view", this);
```

### FakeView 每个方法对应什么

```cpp
class FakeView : public LynxNativeView {
    bool OnCreate() override { return true; }
    // 控件被创建时调用 → DuiLib 的 Init()

    void OnDestroy() override {}
    // 控件被销毁 → DuiLib 的析构

    void OnLayoutChanged(left, top, width, height, pixel_ratio) override {
        // CSS 布局算完了，告诉你要放在哪、多大
        // → DuiLib 的 SetPos(RECT)
        TriggerEvent("resize", ...);  // 通知 JS 布局变了
    }

    void OnMethodInvoked(method, attrs, callback) override {
        // JS 调用你的方法，比如 nativeView.callMethod('play')
        callback(kSuccess, ...);  // 返回结果给 JS
    }
};
```

| 方法 | DuiLib 对应 | 说明 |
|------|-----------|------|
| `OnCreate` | `Init()` | 控件创建时 |
| `OnDestroy` | 析构 | 控件销毁时 |
| `OnLayoutChanged` | `SetPos()` | CSS 布局算完后调用 |
| `OnMethodInvoked` | 自定义消息 | JS 调 C++ 方法的入口 |

---

Day 3 见：[FakeView 实战与架构全景图](/2026-04-20-lynx-source-learning-day3/)
