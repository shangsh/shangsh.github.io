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
builder.SetScreenSize(w, h, dpi)  // 屏幕信息
      .SetFrame(0, 0, w, h)      // 渲染区域
      .SetParent(window_handle_)  // 嵌入 Win32 窗口
      .SetGenericResourceFetcher(...)  // 资源加载器
      .RegisterNativeView<FakeView>(...);  // 注册原生控件
lynx_view_ = builder.Build();
```

### ④ LoadTemplate — 加载模板

空 URL = 加载默认首页。内部读 `resources\homepage\main.lynx.bundle`，这是 Lynx 的打包格式（编译后的 JS 字节码 + CSS + 资源索引）。

### ⑤ 消息循环

标准 Win32 消息循环。Lynx 的渲染和窗口事件跑在同一个线程。


