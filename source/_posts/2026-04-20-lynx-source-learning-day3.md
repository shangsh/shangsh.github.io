---
title: "Lynx 源码学习 Day 3：FakeView 实战与架构全景图"
date: 2026-04-20 00:00:00
tags: [Lynx, C++, Chromium, 源码解析, DuiLib]
categories: 技术分享
---

> 这是 Day 2 的后续，建议先看 [Day 2：ScreenSize、Frame 与 FakeView](/2026-04-20-lynx-source-learning-day2/)

---

## 为什么要 FakeView？

JS+CSS 画不出所有东西：

| JS/CSS 能画的 | 必须用原生控件的 |
|---|---|
| 按钮、文字、图片 | 视频播放器 |
| 列表、滚动 | WebView |
| 动画、渐变 | 地图 |
| 圆角、阴影 | 摄像头预览 |

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
│  LynxEnv → RegisterModule → LynxWindow      │
└──────────────┬──────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────┐
│  LynxView (核心入口)                         │
│                                             │
│  ┌──────────┐  ┌──────────┐  ┌─────────────┐  │
│  │ JS 引擎  │  │ 布局引擎  │  │  渲染引擎   │  │
│  │ QuickJS  │→ │ Lynx     │→ │  Skia 绘制  │  │
│  │ (Lepus)  │  │ Layout   │  │ (CPU/GPU)   │  │
│  └────┬─────┘  └──────────┘  └──────┬──────┘  │
│       │                               │         │
│  ┌────▼─────┐                  ┌─────▼─────┐   │
│  │ NAPI 绑定 │                  │ Win32     │   │
│  │ JS↔C++桥 │                  │ Surface   │   │
│  └──────────┘                  └───────────┘   │
└─────────────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────┐
│  .lynx.bundle (模板包)                      │
│  JS字节码 + CSS + 资源索引                    │
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

---

未完待续，Day 4 见。
