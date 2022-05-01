layout: blog
title: 在 WPF 中获取窗体或控件句柄
date: 2016-12-16 07:05:39
tags:
  - .Net
  - WPF
categories: 
  - 400-编程
  - .Net
---


引入命名空间：

	using System.Windows.Interop;

窗体： 

	IntPtr hwnd = new WindowInteropHelper(this).Handle;

控件： 

	IntPtr hwnd = ((HwndSource)PresentationSource.FromVisual(uielement)).Handle;