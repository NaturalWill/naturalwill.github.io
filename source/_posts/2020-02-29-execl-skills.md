---
title: Excel 技巧
date: 2020-02-29 16:01:24
tags:
  - Excel
categories:
  - 计算机
  - 办公
---

### 校验身份证号是否正确

假设身份证号所在单元格是 `D5`，公式如下：

    =IF(D5="","",(IF(MID("10X98765432",MOD(SUMPRODUCT(MID(D5,ROW(INDIRECT("1:17")),1)*2^(18-ROW(INDIRECT("1:17")))),11)+1,1)=MID(D5,18,18),"正确","错误")))

### Excel 通过 VBA 批量增加行高

Execl 的“自动换行”和“自动调整行高”功能，存在 2 个问题，会导致我们需要手动为每一行调节行高：

1. 两行的字体之间没有空隙，显得密集，看起来不舒服；
2. 单元格里文字有多行的时候，由于大部分打印机的打印质量是 200/300/600 DPI, 而电脑显示质量一般是 96 DPI，所以打印出来后可能有部分单元格内容显示不全。

解决方法：批量调节行高，代码如下：

    Sub AddRowHeight()

    Application.ScreenUpdating = False
    rh = InputBox("请输入待增行高值：", , 10)
    For i = 1 To ActiveSheet.UsedRange.Rows(ActiveSheet.UsedRange.Rows.Count).Row
    If Application.WorksheetFunction.CountA(Rows(i)) > 0 Then
    Rows(i).RowHeight = Rows(i).RowHeight + rh
    End If
    Next i
    Application.ScreenUpdating = True

    End Sub

<!-- more -->

参阅： http://club.excelhome.net/thread-1280692-1-1.html
