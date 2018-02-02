---
title: "另类的 CefSharp.WPF 中文输入问题解决方案"
date: 2017-10-31 11:30:12
tags:
  - WPF
categories: 
  - 计算机
  - 编程
  - .Net
---

## 引言

由于某个需求，需要在原有的 WPF 程序上内嵌浏览器，最终选定 CefSharp.WPF , 但是还是有不少的问题影响着使用体验，比如：最开始遇到的不能输入中文、能输入中文了输入法候选框或右键菜单却跑到屏幕左上角，使用搜狗等第三方输入法不能输入等等。

Google 上逛了很久，也试了很多方法，最后发现一个最简单好用的方案是借用 WinForm 控件。

<!-- more -->

## 正文

新建一个 .Net 4.5.2 以上的 WinForm 程序，修改输出类型为“类库”，然后通过 NuGet 下载最新的 `CefSharp.WinForms` , 新建一个用户控件 `CefControl`, 添加命名空间：

	using CefSharp;
	using CefSharp.WinForms;

添加如下代码：

    public partial class CefControl : UserControl
    {
        ChromiumWebBrowser Default;

        public CefControl(string url)
        {
            InitializeComponent();

            if (!Cef.IsInitialized)
                Cef.Initialize();
            Default = new ChromiumWebBrowser(url);
            // Add it to the form and fill it to the form window.
            this.Controls.Add(Default);
            Default.Dock = DockStyle.Fill;
        }

        public static void Shutdown()
        {
            if (Cef.IsInitialized)
                Cef.Shutdown();
        }
    }

	
在 WPF 项目中使用：

            <WindowsFormsHost>
                <wflib:CefControl/>
            </WindowsFormsHost>

或：
			
            WindowsFormsHost browser = new WindowsFormsHost();
            browser.Child = new CefControl(url);
			
## 相关资料

[解决 CefSharp WPF控件不能使用输入法输入中文的问题]( http://www.cnblogs.com/Starts_2000/p/cefharp-wpf-osr-ime.html )

[CefSharp WPF 中文输入问题解决方法]( http://www.cnblogs.com/wolf-sun/p/6928456.html )
