---
title: 【另辟蹊径】如何在 WPF 实现 MDI 窗口
date: 2017-03-07 16:59:18
tags:
  - ".Net"
---

## 缘起

大概三个月前，我刚刚来到公司实习，老大说我们要做一个与斗鱼主播端功能相近的视频直播软件，界面用 C# 实现，让我来负责这一块，问我该用 WinForm 还是别的来做？ /(ㄒoㄒ)/~~ 当时我不知道怎的就选了 WPF ，最近，我们要为程序加上预览功能，即要播放 YUV/RGB 数据。 然而，从视频设备那里采集到了 YUV 数据后，转为 Bitmap ，然后再转为 BitmapSource ，到最终呈现出来，非常低效。

这几天，我们发现通过 SDL 进行渲染能有效的降低 CPU 使用率。

然而，坑还是有的，我把 WPF Image 对象和图片数据传给 SDL ，它却直接把整个窗体霸占了 T_T 。

因此，只能把整个窗体给 SDL 了。

<!-- more -->

## 网络上的 WPF MDI 方案

先说说我在网上找到的 MDI 方案。

### 方案一  

用Host的方式，将一个窗体的句柄设置为另一窗体的子窗体 （调用 SetParent API），大概是这样子的：

	using System;
	using System.Runtime.InteropServices;
	using System.Windows;
	using System.Windows.Input;
	using System.Windows.Interop;

	namespace WpfMdiDemo
	{
		/// <summary>
		/// MainWindow.xaml 的交互逻辑
		/// </summary>
		public partial class MainWindow : Window
		{
			[DllImport("user32.dll", EntryPoint = "SetParent")]
			public static extern IntPtr SetParent(IntPtr childPtr, IntPtr parentPtr);
			
			public MainWindow()
			{
				InitializeComponent();
			}

			private ChildWindow c;
			private void MainWindow_OnLoaded(object sender, RoutedEventArgs e)
			{
				c = new ChildWindow{ Owner = this };
				c.Show();
				
				// 将一个窗体的句柄设置为另一窗体的子窗体，这句一定得在 Window.Show() 之后，不然不会有效
				WindowInteropHelper parentHelper = new WindowInteropHelper(this);
				WindowInteropHelper childHelper = new WindowInteropHelper(c);
				SetParent(childHelper.Handle, parentHelper.Handle);
			}
		}
	}
	
参考：[WPF实现MDI窗口，并解决花屏问题](http://blog.csdn.net/qing2005/article/details/6523721)
	
	

### 方案二 

这是一个开源的 WPF MDI 解决方案。 [WPF Multiple Document Interface (MDI)](http://wpfmdi.codeplex.com/)


## 我的解决方案

方案二虽然据说很好的实现了 WPF MDI ，但是它的 MDI 实际上还是通过控件的形式实现的，传递给 SDL 的话， SDL 还是会把我的 MainWindow 吃掉，并不符合我的需求。

So, 我只能基于方案一打造我的 MDI 了。

### 方案一中遇到的问题

* 调用 SetParent 后，子窗体的外观会变得像 WinForm 窗体而与主窗体风格迥异
* 调用 SetParent 后，子窗体会闪烁一下，再出现在父窗体中。

### 解决方法

1. 隐藏窗口标题栏、边框，隐藏任务栏图标，禁止手动调整大小：

	WindowStyle="None" ShowInTaskbar="False" ResizeMode="NoResize" 

2. 预先设置好子窗体的位置，先将宽高设置为 1 ，待调用 SetParent 后，还原宽高：


	// c 为 ChildWindow 
	// gPreview 是父窗体中的一控件， c 的初始位置根据 gPreview 定位
	// w, h 是 c 的实际宽高
	Window window = Window.GetWindow(gPreview);
	var point = gPreview.TransformToAncestor(window).Transform(new Point(0, 0));
	
	c.Left = point.X + (gPreview.ActualWidth - w) / 2;
	c.Top = point.Y + (gPreview.ActualHeight - h) / 2;
	c.Width = 1;
	c.Height = 1;

	c.Show();
	
	WindowInteropHelper parentHelper = new WindowInteropHelper(_main);
	WindowInteropHelper childHelper = new WindowInteropHelper(c);
	SetParent(childHelper.Handle, parentHelper.Handle);

	c.Width = w;
	c.Height = h;
	
### 扩展

如果需要让窗口的显示标题栏及边框，可以通过在主窗体增加仿标题及边框的控件，然后子窗体根据该控件调整位置与大小即可。
