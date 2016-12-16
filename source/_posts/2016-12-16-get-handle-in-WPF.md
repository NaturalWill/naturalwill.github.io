layout: blog
title: 在 WPF 中获取窗体或控件句柄
date: 2016-12-16 07:05:39
tags:
  - C#
  - notes
---


窗体： 

	IntPtr hwnd = new WindowInteropHelper(this).Handle;

控件： 

	IntPtr hwnd = ((HwndSource)PresentationSource.FromVisual(uielement)).Handle;