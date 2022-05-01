---
title: Excel 技巧
date: 2020-02-29 16:01:24
tags:
  - Excel
categories:
  - 100-办公软件使用

---

### 校验身份证号是否正确

假设身份证号所在单元格是 `D5`，公式如下：

    =IF(D5="","",(IF(MID("10X98765432",MOD(SUMPRODUCT(MID(D5,ROW(INDIRECT("1:17")),1)*2^(18-ROW(INDIRECT("1:17")))),11)+1,1)=MID(D5,18,18),"正确","错误")))

### Excel VBA 

#### 批量增加行高

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


#### 完整显示跨页合并的单元格

    Sub ShowALL()
    Dim P
    Dim MergeAddress As String
    Dim PageCell As Range
    Dim MergeValue
    ActiveWindow.View = xlPageBreakPreview
    For Each P In ActiveSheet.HPageBreaks
    Set PageCell = Cells(P.Location.Row - 1, ActiveCell.Column)
    If PageCell.MergeCells And Not Intersect(Cells(P.Location.Row, ActiveCell.Column), PageCell.MergeArea) Is Nothing Then
    MergeAddress = PageCell.MergeArea.Address
    MergeValue = PageCell.MergeArea(1).Value
    PageCell.MergeArea.UnMerge
    Range(Range(MergeAddress)(1), PageCell).Merge
    With Range(PageCell.Offset(1, 0), Cells(Split(MergeAddress, "$")(4), ActiveCell.Column))
    .Merge
    .Value = MergeValue
    End With
    End If
    Next
    ActiveWindow.View = xlNormalView
    End Sub


#### 保留表头拆分数据为若干新工作簿

    Sub 保留表头拆分数据为若干新工作簿()
    Dim arr, d As Object, k, t, i&, lc%, rng As Range, c%
    c = Application.InputBox("请输入拆分列号", , 4, , , , , 1)
    If c = 0 Then Exit Sub
    Application.ScreenUpdating = False
    Application.DisplayAlerts = False
    arr = [a1].CurrentRegion
    lc = UBound(arr, 2)
    Set rng = [a1].Resize(, lc)
    Set d = CreateObject("scripting.dictionary")
    For i = 2 To UBound(arr)
    If Not d.Exists(arr(i, c)) Then
    Set d(arr(i, c)) = Cells(i, 1).Resize(1, lc)
    Else
    Set d(arr(i, c)) = Union(d(arr(i, c)), Cells(i, 1).Resize(1, lc))
    End If
    Next
    k = d.Keys
    t = d.Items
    For i = 0 To d.Count - 1
    With Workbooks.Add(xlWBATWorksheet)
    rng.Copy .Sheets(1).[a1]
    t(i).Copy .Sheets(1).[a2]
    .SaveAs Filename:=ThisWorkbook.Path & "\" & k(i) & ".xlsx"
    .Close
    End With
    Next
    Application.DisplayAlerts = True
    Application.ScreenUpdating = True
    MsgBox "完毕"
    End Sub


#### 拆分并填充单元格

    Sub 拆分并填充单元格()
    '
    ' 拆分并填充单元格 宏
    '
    ' 可以将选中的所有单元格都拆开并填充
    '
    If Application.Selection.MergeCells = True Then
    Set selectedRange = Application.Selection
    selectedRange.UnMerge
    selectedRange.Value = selectedRange.Cells(1, 1).Value
    Else
    For Each selectedCell In Application.Selection
    If selectedCell.MergeCells = True Then
    Set selectedRange = selectedCell.MergeArea
    selectedRange.UnMerge
    selectedRange.Value = selectedRange.Cells(1, 1).Value
    End If
    Next
    End If
    End Sub




