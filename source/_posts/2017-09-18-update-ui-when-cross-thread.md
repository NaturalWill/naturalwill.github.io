---
layout: blog
title: C# 跨线程更新 UI
date: 2017-09-18 18:01:28
tags:
  - ".Net"
categories: 
  - 400-编程
  - .Net
---

## Winforms 跨线程更新 UI

在 Winforms 中, 所有的控件都包含 `InvokeRequired` 属性, 如果我们要更新UI，通过它我们可以判断是否需要调用 `[Begin]Invoke`.


### 直接使用

	delegate void SetTextCallback(string text);

	public void SetText(string text)
	{
		if (InvokeRequired)
		{
			var d = new SetTextCallback(SetText);
			this.textBox1.Invoke(d, new object[] { text });
		}
		else
		{
			this.textBox1.Text = text;
		}
	}

直接调用 `SetText` 即可。

<!-- more -->

### 使用扩展方法

    public static class MyClass
    {
		public static void InvokeIfRequired(this ISynchronizeInvoke obj, MethodInvoker action)
		{
			if (obj.InvokeRequired)
			{
				var args = new object[0];
				obj.Invoke(action, args);
			}
			else
			{
				action();
			}
		}
	}

使用：

	this.textBox1.InvokeIfRequired(() =>
	{
		// Do anything you want with the control here
		this.textBox1.Text = "text";
	});


## WPF 跨线程更新 UI

在 WPF 中，与 WinForms 中 `InvokeRequired` 属性相对应的有： `DispatcherObject.CheckAccess()` 和 `Dispatcher.CheckAccess()`.

	// Uses the Dispatcher.CheckAccess method to determine if 
	// the calling thread has access to the thread the UI object is on.
	private void TryToUpdateButtonCheckAccess(object uiObject)
	{
		Button theButton = uiObject as Button;

		if (theButton != null)
		{
		    // Checking if this thread has access to the object.
		    if (theButton.Dispatcher.CheckAccess())
		    {
		        // This thread has access so it can update the UI thread.
		        UpdateButtonUI(theButton);
		    }
		    else
		    {
		        // This thread does not have access to the UI thread.
		        // Place the update method on the Dispatcher of the UI thread.
		        theButton.Dispatcher.BeginInvoke(DispatcherPriority.Normal,
		            new UpdateUIDelegate(UpdateButtonUI), theButton);
		    }
		}
	}

然而, `CheckAccess` 和 `VerifyAccess` 被标记为永远不在智能提示在显示，是否在暗示我们应该直接使用 `Dispatcher.Invoke()` 呢？

    //
    // 摘要:
    //     确定调用线程是否为与此 System.Windows.Threading.Dispatcher 关联的线程。
    //
    // 返回结果:
    //     如果调用线程是与此 System.Windows.Threading.Dispatcher 关联的线程，则为 true；否则为 false。
    [EditorBrowsable(EditorBrowsableState.Never)]
    public bool CheckAccess();

相关链接：

[ Correct usage (or not-usage) of Dispatcher.CheckAccess() ](https://stackoverflow.com/questions/12937902/correct-usage-or-not-usage-of-dispatcher-checkaccess )

