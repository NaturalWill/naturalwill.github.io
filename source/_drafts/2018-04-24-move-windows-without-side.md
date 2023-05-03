---
title: "C# 无边框窗体移动"
date: 2018-04-16 13:54:07
categories:
  - 计算机
  - 编程
  - .Net
---

用到的命名空间

```csharp
using System;
using System.IO;
using System.Runtime.InteropServices;
```

<!-- more -->

代码

```csharp
private void Grid_MouseLeftButtonDown(object sender, MouseButtonEventArgs e)
{
    if (e.LeftButton == MouseButtonState.Pressed)
    {
        DragMove();
    }
}
```

```csharp
protected override void OnMouseMove(MouseEventArgs e)
{
    base.OnMouseMove(e);

    if (e.LeftButton == MouseButtonState.Pressed &&
        // For some reason Slider doesn't seem to mark MouseMove events as handled.
        VisualTreeHelper.HitTest(_opacitySlider, e.GetPosition(_opacitySlider)) == null)
    {
        DragMove();
    }
}

```