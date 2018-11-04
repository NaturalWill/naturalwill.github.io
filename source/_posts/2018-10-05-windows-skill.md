---
title: Windows 小技巧
date: 2018-10-05 18:02:20
tags:
---

## 文件夹别名

在使用 Windows 系统时，会发现部分英文路径中的目录显示的却是中文名（如：显示为“我的文档”真实的文件名为 Documents ）。如果我们想建立一个目录放文件，需要使用英文路径，但是为了美观，用中文别名，这又是怎么实现的呢？

在当前目录新建一文件 `desktop.ini` ，修改其内容如下：

    [.ShellClassInfo]
    LocalizedResourceName=显示名称

由于系统缓存的关系，文件名未及时改变。用下面的方法让它立刻生效。

右击文件夹 -> 属性 -> 自定义 -> 更改图标；随便选一个图标，然后点击确定。

这时会发现文件名已变成我们要设置的名称，我们刚刚创建的 `desktop.ini` ，会被设置为“受系统保护的文件”并隐藏起来。

重新打开 `desktop.ini` ，会发现里面多了一行用来描述图标的。

    [.ShellClassInfo]
    LocalizedResourceName=显示名称
    IconResource=C:\Windows\system32\SHELL32.dll,7

如需恢复默认，可按如下操作恢复。

右击文件夹 -> 属性 -> 自定义 -> 更改图标 -> 还原默认值-> 确定。

<!-- more -->

## 激活 Windows 企业版

    # 设置 kms 服务器
    slmgr /skms your.kms.server

    # 执行激活命令
    slmgr /ato

其他命令

    # 检查系统版本及激活信息
    slmgr /dlv

    # 安装激活密钥
    slmgr /ipk XXXXX-XXXXX-XXXXX-XXXXX-XXXXX

## 激活 Office VL 版

> Office16 == Office 2016
> Office15 == Office 2013
> Office14 == Office 2010

    cd "C:\Program Files\Microsoft Office\Office16"
    cscript ospp.vbs /sethst:your.kms.server
    cscript ospp.vbs /act

如果是 64 位操作系统安装了 32 位的 Office ，第一条命令改为

    cd "C:\Program Files (x86)\Microsoft Office\Office16"

其他命令

    # 查看原密钥后五位
    cscript ospp.vbs /dstatus

    # 卸载原密钥，XXXXX代表后五位
    cscript ospp.vbs /unpkey:XXXXX

    # 安装新密钥
    cscript ospp.vbs /inpkey:XXXXX-XXXXX-XXXXX-XXXXX-XXXXX

## 注释

### CMD 注释

CMD 注释形式如下：

1. :: 注释内容（第一个冒号后也可以跟任何一个非字母数字的字符）
2. REM 注释内容（不能出现重定向符号和管道符号）
3. %注释内容%（可以用作行间注释，不能出现重定向符号和管道符号）
4. :标签 注释内容（可以用作标签下方段的执行内容）

### PowerShell 注释

PowerShell 的注释符分为行注释符和块注释符。行注释符使用 “井号（`#`）”引起一行；块注释符使用“`<#`”和“`#>`”来引起一段注释。

行注释符，举例如下：

    # 定义一个计数变量
    $i = 0

块注释符、多行注释，举例如下：

    <#
        文件：xxx.ps1
        用途：用于测试的xxx功能脚本
    #>
