---
title: "Lynx 源码学习 Day 1：从 main 到屏幕像素"
date: 2026-04-20 00:00:00
tags: [Lynx, C++, Chromium, 源码解析, DuiLib]
categories: 技术分享
---

> 作者背景：C++ PC 客户端老兵，想用 Lynx 替代 DuiLib，用 JS 写 PC 客户端 UI。

---

## 一、入口在哪里

Lynx Explorer 的 `main.cc` 只有 25 行，非常干净：

```cpp
int APIENTRY wWinMain(HINSTANCE instance, ...) {
    auto& lynx_env = lynx::pub::LynxEnv::GetInstance();  // ① 初始化环境
    lynx_env.SetDevtoolEnabled(true);

    lynx_env.RegisterNativeModule("ExplorerModule",  // ② 注册原生模块
                                  ExplorerModuleCreator, nullptr);

    auto* window = new LynxWindow(0, 0, 800, 600);  // ③ 创建窗口
    window->SetQuitOnClose(true);
    window->LoadTemplate("");  // ④ 加载模板

    ::MSG msg;  // ⑤ 消息循环
    while (::GetMessage(&msg, nullptr, 0, 0)) {
        ::TranslateMessage(&msg);
        ::DispatchMessage(&msg);
    }
}
```

五步走，一步步拆。

### ① LynxEnv — 全局环境

单例，全局只初始化一次。开了 devtool 方便调试。

类比 DuiLib：类似 `CPaintManagerUI::GetInstance()`。

### ② RegisterNativeModule — JS↔C++ 桥梁

JS 里写 `require('ExplorerModule')` 就能调 C++ 方法。这是 JS 调 C++ 的入口，以后会自己写很多。

### ③ LynxWindow — 窗口 + 引擎

`LynxWindow` 构造函数做了两件大事：

1. 创建 Win32 窗口（RegisterClass + CreateWindow，跟 DuiLib 一样）
2. 创建 LynxView（核心渲染引擎）

关键代码在 Builder 模式：

```cpp
LynxView::Builder builder;
builder.SetScreenSize(w, h, dpi)      // 屏幕信息
      .SetFrame(0, 0, w, h)          // 渲染区域
      .SetParent(window_handle_)       // 嵌入 Win32 窗口
      .SetGenericResourceFetcher(...) // 资源加载器
      .RegisterNativeView<FakeView>(...);  // 注册原生控件
lynx_view_ = builder.Build();
```

### ④ LoadTemplate — 加载模板

空 URL = 加载默认首页。内部读 `resources\homepage\main.lynx.bundle`，这是 Lynx 的打包格式（编译后的 JS 字节码 + CSS + 资源索引）。

### ⑤ 消息循环

标准 Win32 消息循环。Lynx 的渲染和窗口事件跑在同一个线程。

---

## 二、ScreenSize vs Frame — 窗口和视口的区别

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

**大多数情况下 ScreenSize 和 Frame 宽高一样**（Lynx 占满窗口），只有嵌入部分区域时才不同。

|        | ScreenSize | Frame                  |
|--------|------------|------------------------|
| 描述什么 | 屏幕的物理信息 | Lynx 渲染区域          |
| DuiLib 类比 | `GetSystemMetrics()` + DPI | `MoveWindow()` |
| 给谁用  | JS 引擎算像素 | 渲染引擎画图           |

---

## 三、FakeView — 自定义原生控件

### 为什么要 FakeView？

JS+CSS 画不出所有东西：

| JS/CSS 能画的 | 必须用原生控件的 |
|---|---|
| 按钮、文字、图片 | 视频播放器 |
| 列表、滚动 | WebView |
| 动画、渐变 | 地图 |
| 圆角、阴影 | 摄像头预览 |

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

    void OnLayoutChanged(float l, float t, float w, float h, float dpi) override {
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

### 实战：做一个视频播放器控件

```cpp
// C++ 端
class VideoPlayer : public LynxNativeView {
    HWND video_hwnd_;

    bool OnCreate() override {
        video_hwnd_ = CreateWindow(L"video_player", ...);
        return true;
    }

    void OnLayoutChanged(float l, float t, float w, float h, float dpi) override {
        MoveWindow(video_hwnd_, l*dpi, t*dpi, w*dpi, h*dpi, TRUE);
    }

    void OnMethodInvoked(const char* method, ..., callback) override {
        if (strcmp(method, "play") == 0) {
            PlayVideo();
            callback(kSuccess, ...);
        }
    }
};

// 注册
builder.RegisterNativeView<VideoPlayer>("x-video-player", nullptr);
```

```jsx
// JS 端
<x-video-player style="width: 100%; height: 300px;" ref="player" />
this.refs.player.callMethod('play')  // 调 C++ 的 PlayVideo()
```

**一句话：FakeView = DuiLib 里的自定义控件，`RegisterNativeView` 就是告诉 Lynx "这个标签归我管"。**

---

## 四、架构全景图

```
┌─────────────────────────────────────────────┐
│  main.cc                                    │
│  LynxEnv → RegisterModule → LynxWindow     │
└──────────────┬──────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────┐
│  LynxView (核心入口)                         │
│                                             │
│  ┌──────────┐  ┌──────────┐  ┌─────────────┐ │
│  │ JS 引擎  │  │ 布局引擎  │  │  渲染引擎   │ │
│  │ QuickJS  │→ │ Lynx     │→ │  Skia 绘制  │ │
│  │ (Lepus)  │  │ Layout   │  │ (CPU/GPU)  │ │
│  └────┬─────┘  └──────────┘  └──────┬──────┘ │
│       │                               │         │
│  ┌────▼─────┐                  ┌─────▼─────┐  │
│  │ NAPI 绑定 │                  │ Win32     │  │
│  │ JS↔C++桥 │                  │ Surface   │  │
│  └──────────┘                  └───────────┘  │
└─────────────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────┐
│  .lynx.bundle (模板包)                     │
│  JS字节码 + CSS + 资源索引                   │
└─────────────────────────────────────────────┘
```

### 完整数据流

```
main → 初始化Lynx环境 → 创建Win32窗口 → Builder构建LynxView（设为子窗口）
     → 设置资源路径 → 加载 .lynx.bundle → JS引擎编译执行 → 生成Element树
     → 布局引擎计算位置大小 → 渲染引擎生成绘制指令 → Skia画到窗口
```

---

## 五、待解答的疑问

1. ❓ Engine 线程具体怎么和 UI 线程通信？Actor 模式是什么？
2. ❓ `.lynx.bundle` 的内部结构是什么？怎么打包的？
3. ❓ LynxView 内部 LynxTemplateRenderer 和 LynxUIRenderer 怎么分工？
4. ❓ 如何在自己的 Win32 窗口里嵌入 LynxView（不用 Explorer）？
