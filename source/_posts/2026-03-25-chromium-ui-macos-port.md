---
title: Chromium UI框架 macOS移植完全指南 - 从Windows到Apple Silicon
date: 2026-03-25 00:15:00
tags: [Chromium, C++, macOS, UI框架, 跨平台, 移植]
categories: 技术分享
---

# Chromium UI框架 macOS移植完全指南

> 作为一个10年PC客户端老兵，带你搞定Chromium UI的跨平台移植

## 背景

之前我们实现了[《手写一个极简Chromium浏览器》](https://github.com/shangsh/chromium-mini-browser)，很多同学对Chromium的架构有了初步认识。今天来点更深入的——把整个 **Chromium UI (chromium_ui)** 框架移植到 macOS！

## Chromium UI 是什么？

Chromium UI 是 Chromium 浏览器的UI层框架，包含：

```
┌─────────────────────────────────────────────────────────────┐
│                     chromium_ui 架构                         │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────┐    │
│  │                   View Framework                     │    │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐  │    │
│  │  │  View   │ │ Window  │ │ Widget  │ │ Control│  │    │
│  │  │  视图   │ │  窗口   │ │  控件   │ │  组件  │  │    │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘  │    │
│  └─────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                    gfx (Graphics)                   │    │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐  │    │
│  │  │ Bitmap  │ │ Canvas  │ │  Font   │ │  Path  │  │    │
│  │  │ 位图    │ │ 画布    │ │  字体   │ │  路径  │  │    │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘  │    │
│  └─────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                    base (基础库)                    │    │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐  │    │
│  │  │ Thread │ │  Time   │ │ Atomic  │ │ String │  │    │
│  │  │ 线程   │ │  时间   │ │ 原子操作 │ │ 字符串  │  │    │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘  │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### 核心组件

| 组件 | 功能 | Windows实现 |
|------|------|------------|
| View Framework | UI组件系统 | Win32控件 |
| gfx/Graphics | 图形绘制 | GDI/GDI+ |
| base | 基础库 | Windows API |
| animation | 动画系统 | Win32定时器 |

## 移植挑战

最大的问题是：**Windows依赖太深**！

```cpp
// 原始代码 (Windows only)
#include <windows.h>  // ❌ macOS没有

// 各种Windows类型
typedef LONG Atomic32;      // ❌
typedef DWORD ColorRef;    // ❌
HWND hwnd;                // ❌
```

### 主要障碍

1. **Windows头文件** - 几百个文件include `<windows.h>`
2. **GDI+图形库** - 完全Windows特有
3. **原子操作** - InterlockedCompareExchange 等
4. **消息循环** - MSG、WNDPROC 等
5. **COM接口** - IUnknown、IClassFactory 等

## 解决方案：条件编译 + 独立实现

### 策略一：条件编译 (#ifdef)

```cpp
// base/atomicops.h
#ifdef _WIN32
    // Windows实现
    typedef LONG Atomic32;
#else
    // macOS实现 - 用GCC内置原子操作
    typedef int32_t Atomic32;
    
    inline Atomic32 NoBarrier_CompareAndSwap(...) {
        return __sync_val_compare_and_swap(ptr, old_value, new_value);
    }
#endif
```

### 策略二：独立实现 (Aura_mac)

```cpp
// aura_mac/view/view_mac.h
#ifdef AURA_MAC
#import <Cocoa/Cocoa.h>

namespace aura {
namespace mac {

class ViewMac {
    NSView* nsview_;  // ✅ 换成AppKit
    void SetBounds(int x, int y, int w, int h);
};

} // namespace mac
} // namespace aura
#endif
```

## 移植实现

### 1. 基础库移植

#### atomicops.h - 原子操作

```cpp
// macOS使用GCC内置原子操作
inline Atomic32 NoBarrier_CompareAndSwap(volatile Atomic32* ptr,
    Atomic32 old_value, Atomic32 new_value) 
{
    return __sync_val_compare_and_swap(ptr, old_value, new_value);
}

inline Atomic32 Barrier_AtomicIncrement(volatile Atomic32* ptr, Atomic32 increment) {
    return __sync_add_and_fetch(ptr, increment);
}
```

#### time.h - 时间处理

```cpp
// 添加跨平台类型定义
#ifndef _WIN32
#include <sys/time.h>
#include <cstdint>

typedef uint64_t QWORD;
typedef uint32_t DWORD;
typedef struct _FILETIME {
    DWORD dwLowDateTime;
    DWORD dwHighDateTime;
} FILETIME;
#endif
```

### 2. UI框架移植

#### View类 (macOS实现)

```cpp
// aura_mac/view/view_mac.h
class ViewMac {
public:
    void SetBounds(int x, int y, int width, int height) {
        if (nsview_) {
            [nsview_ setFrame:NSMakeRect(x, y, width, height)];
        }
    }
    
    void SetVisible(bool visible) {
        if (nsview_) {
            [nsview_ setHidden:!visible];
        }
    }
    
private:
    NSView* nsview_;  // AppKit视图
};
```

#### Window类 (macOS实现)

```cpp
// aura_mac/window/window_mac.h
class WindowMac {
public:
    bool Init() {
        window_ = [[NSWindow alloc] 
            initWithContentRect:NSMakeRect(0, 0, 800, 600)
            styleMask:NSTitledWindowMask | NSClosableWindowMask
            backing:NSBackingStoreBuffered
            defer:NO];
        [window_ center];
        return window_ != nil;
    }
    
private:
    NSWindow* window_;
};
```

### 3. 控件移植

#### Button按钮

```cpp
class ButtonMac : public ViewMac {
public:
    void SetText(const std::wstring& text) {
        if (button_) {
            NSString* s = [NSString stringWithUTF8String:
                std::string(text.begin(), text.end()).c_str()];
            [button_ setTitle:s];
        }
    }
    
private:
    NSButton* button_;  // AppKit按钮
};
```

#### Label标签

```cpp
class LabelMac : public ViewMac {
    void SetFont(NSFont* font) {
        if (label_ && font) {
            [label_ setFont:font];
        }
    }
    
    void SetTextColor(NSColor* color) {
        if (label_ && color) {
            [label_ setTextColor:color];
        }
    }
    
private:
    NSTextField* label_;
};
```

## 完整控件清单

| 控件 | macOS实现 | 状态 |
|------|----------|------|
| View | NSView | ✅ |
| Window | NSWindow | ✅ |
| Widget | NSView+NSWindow | ✅ |
| Button | NSButton | ✅ |
| Label | NSTextField | ✅ |
| TextField | NSTextField | ✅ |
| Checkbox | NSButton(checkbox) | ✅ |
| ComboBox | NSPopUpButton | ✅ |
| ProgressBar | NSProgressIndicator | ✅ |
| Slider | NSSlider | ✅ |
| TabControl | NSTabView | ✅ |
| Menu | NSMenu | ✅ |
| Toolbar | NSToolbar | ✅ |
| StatusBar | NSView | ✅ |
| TreeView | NSOutlineView | ✅ |

## 测试效果

```
═══════════════════════════════════════════════════════════════
   Chromium Aura UI - macOS Complete Port 🦞
═══════════════════════════════════════════════════════════════
   Window: 1100x750
   Completed: View, Widget, Button, Label, TextField
   Completed: Checkbox, ComboBox, ProgressBar, Slider
   Completed: TabControl, Menu, Toolbar, StatusBar
   Completed: Dialog, TreeView
═══════════════════════════════════════════════════════════════
```

## 关键技巧

### 1. 宏开关隔离

```cpp
// 在CMakeLists.txt中
if(APPLE)
    set(CMAKE_CXX_FLAGS "-DAURA_MAC")
endif()

// 在代码中
#ifdef AURA_MAC
    #import <Cocoa/Cocoa.h>
    // macOS专用代码
#else
    #include <windows.h>
    // Windows代码
#endif
```

### 2. 类型桥接

```cpp
// 创建桥接头文件
// base/win_types.h
#ifndef _WIN32
typedef void* HWND;
typedef void* HDC;
typedef void* HICON;
// ...
#endif
```

### 3. 逐步替换

不要试图一次性移植所有代码，而是：
1. 先让基础库编译通过
2. 再逐个移植UI控件
3. 最后整合测试

## 代码结构

```
chromium_ui/
├── CMakeLists.txt              # 主构建(跨平台)
├── base/
│   ├── atomicops.h            # 原子操作 ✅
│   ├── atomicops_mac.h       # macOS实现
│   ├── time.h                # 时间处理 ✅
│   └── win_types.h           # Windows类型桥接
├── gfx/
│   ├── point.h, rect.h      # 图形基础 ✅
│   └── ...
├── view_framework/
│   ├── view_mac.h/cpp        # View (macOS)
│   ├── window_mac.h/mm       # Window (macOS)
│   ├── widget_mac.h/mm       # Widget (macOS)
│   └── controls/
│       ├── button_mac.h/cpp  # Button
│       ├── label_mac.h/cpp   # Label
│       └── textfield_mac.h/cpp
└── aura_mac/                 # 独立macOS实现
    ├── view/
    ├── window/
    └── controls/
```

## 编译运行

```bash
# 克隆项目
git clone https://github.com/shangsh/chromium_ui.git
cd chromium_ui

# macOS编译
mkdir build && cd build
cmake ..
make

# 运行测试
../test_complete/test_complete
```

## 总结

通过这次移植，我们学会了：

1. **条件编译** - 用 `#ifdef` 实现跨平台
2. **抽象接口** - 把平台相关代码抽象出来
3. **逐步移植** - 从基础库到UI层
4. **宏开关控制** - 不影响原有Windows编译

chromium_ui现在可以在Windows和macOS双平台编译运行！

---

**项目地址**: https://github.com/shangsh/chromium_ui

**测试Demo**: 
- test_complete - 完整控件测试
- test_interactive - 交互测试
- test_advanced - 高级功能

*有问题欢迎评论区交流~*
