---
title: 那些年，在 .Net 里踩过的坑
date: 2018-02-07 16:48:16
tags:
  - WPF
categories: 
  - 400-编程
  - .Net
---


## WPF 与 WinForm 混用

### AllowsTransparency & WindowsFormsHost

WPF 启用窗体透明(`AllowsTransparency="true"`) 后， `WindowsFormsHost` 的内容不可见。

<!-- more -->
#### 原因

这是一个 Win32 的限制。

使用 `WS_EX_LAYERED` 和 `UpdateLayeredWindow()` 的分层窗体不支持子窗体。

要支持子窗口需要使用不变的不透明度 (`WS_EX_LAYERED` and `SetLayeredWindowAttributes()`) 。

#### 解决方法

一个较好的解决方案，见 [webbrowser-control-on-transparent-wpf-window][webbrowser-control-on-transparent-wpf-window]。

感谢 [Fredrik Hedblad][WPF WindowsFormsHost is not visible when AllowsTransparency=“True”] 。

#### More

* [Extended Window Styles][Extended Window Styles]



[Extended Window Styles]: https://msdn.microsoft.com/en-us/library/windows/desktop/ff700543(v=vs.85).aspx
[WPF WindowsFormsHost is not visible when AllowsTransparency=“True”]: https://stackoverflow.com/questions/4108531/wpf-windowsformshost-is-not-visible-when-allowstransparency-true 


[webbrowser-control-on-transparent-wpf-window]: https://blogs.msdn.microsoft.com/changov/2009/01/19/webbrowser-control-on-transparent-wpf-window/









