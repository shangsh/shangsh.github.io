---
title: Lynx Explorer Windows 手动编译指南
date: 2026-04-19 23:00:00
tags: [Lynx, Chromium, C++, Windows, 编译]
categories: 技术分享
---

# Lynx Explorer Windows 手动编译指南

> 手动编译 Lynx Explorer 全流程记录，适用于 Windows + VS2022 环境

## 1. 环境准备

安装必要工具链：

- VS2022 Community
- Windows SDK 10
- Git
- Node.js 22+

每次开新终端都需要执行以下环境变量设置：

```powershell
$env:DEPOT_TOOLS_WIN_TOOLCHAIN=0
$env:GYP_MSVS_OVERRIDE_PATH="C:\Program Files\Microsoft Visual Studio\2022\Community"
$env:WINDOWSSDKDIR="C:\Program Files (x86)\Windows Kits\10"
```

## 2. 同步依赖

```powershell
cd H:\open\lynx

# 用 hab 同步第三方依赖（耗时较长）
powershell -ExecutionPolicy Bypass -File .\hab.ps1 sync . --target clay
```

## 3. 确认构建工具存在

```powershell
# gn 和 ninja 应该在这两个位置
ls .\buildtools\gn\gn.exe
ls .\buildtools\ninja\ninja.exe
```

如果没有，需要手动下载（通过代理）：

```powershell
# gn
$env:HTTPS_PROXY="http://10.19.2.146:62677"
# 从 https://github.com/lynx-family/buildtools/releases 下载对应版本
# 解压到 buildtools/gn/

# ninja
# 从 https://github.com/ninja-build/ninja/releases 下载
# 解压到 buildtools/ninja/
```

## 4. 生成构建文件

```powershell
.\buildtools\gn\gn.exe gen out\Default --ide=vs --args='
desktop_enable_embedder_layer=true
enable_clay_standalone=true
disable_visibility_hidden=true
use_ndk_static_cxx=false
enable_linker_map=false
enable_clay=true
is_headless=true
skia_enable_flutter_defines=true
skia_use_dng_sdk=false
skia_use_sfntly=false
skia_enable_pdf=false
skia_enable_svg=true
enable_svg=true
skia_enable_skottie=true
skia_use_x11=false
skia_use_wuffs=true
skia_use_expat=true
skia_use_fontconfig=false
clay_enable_skshaper=true
skia_use_icu=true
allow_deprecated_api_calls=true
stripped_symbols=true
is_official_build=true
enable_lto=false
is_clang=true
enable_lepusng_worklet=true
enable_napi_binding=true
is_debug=false
enable_inspector=true
enable_libcpp_abi_namespace_cr=true
jsengine_type="quickjs"
'
```

`--ide=vs` 会同时生成 VS 解决方案，方便调试。

## 5. 编译

```powershell
.\buildtools\ninja\ninja.exe -C out\Default explorer
```

产出物：

- `out\Default\lynx_explorer\lynx_explorer.exe` — 主程序
- `out\Default\lynx_explorer\lynx.dll` — 核心引擎

## 6. 用 VS 调试

`--ide=vs` 会生成 `out\Default\all.sln`，双击打开就行：

```powershell
start out\Default\all.sln
```

在 VS 里：

1. 右键 `lynx_explorer` 项目 → 设为启动项目
2. F5 开始调试（或 Ctrl+F5 不调试运行）
3. 断点随便打，C++ 代码都能断

## ⚠️ 踩坑记录

### 坑1：Python 子进程失败

`.venv\bin\python3.exe` 可能无法 spawn 子进程。修改 `devtool\base_devtool\resources\copy_resources.py`，把 `"python3"` 改成 `sys.executable`：

```python
# 原来的
command = ["python3", script, output]

# 改成
command = [sys.executable, script, output]
```

### 坑2：`.venv\bin\python3.exe` 损坏

如果 `.venv\bin\python3.exe` 不存在或无法运行子进程，从 Scripts 目录复制：

```powershell
Copy-Item .venv\Scripts\python3.exe .venv\bin\python3.exe -Force
```

### 坑3：`.gn` 中 script_executable 路径

确认 `.gn` 文件里的 python 路径有 `.exe` 后缀：

```
script_executable = rebase_path("//.venv/bin/python3.exe")
```

### 坑4：hab sync 会清空 buildtools

每次 `hab sync` 后 gn 和 ninja 可能被清掉，需要重新放回去。

---

有什么不清楚的欢迎留言讨论。
