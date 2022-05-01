---
title: Visual Studio 技巧
date: 2018-04-03 09:57:52
tags:
  - NuGet
  - VisualStudio
categories: 
  - 400-软件使用
  - Windows
---

## 重新安装 NuGet 包

有时候打开项目，发现 NuGet 包全部丢失了，点击`还原 NuGet 包`也没有任何作用。

这时我们可以点击`工具`->`NuGet 包管理器`->`程序包管理控制台`打开控制台，在里面输入以下命令重新安装解决方案的 NuGet 包。

    Update-Package -Reinstall

也可以只重新安装某一项目的 NuGet 包。

    Update-Package -ProjectName "DemoProject" -Reinstall

## DXERR.LIB

**VS 2015 / 2017:**The VS 2015 / 2017 C Runtime is not compatible with the`DXERR.LIB`that ships in the legacy DirectX SDK. You will get link errors trying to use it. You can use this module to replace DXERR LIB but will have to rebuild the code that uses it. You can try linking with`legacy_stdio_definitions.lib`instead, but ideally you'd remove this dependency on the legacy DirectX SDK.