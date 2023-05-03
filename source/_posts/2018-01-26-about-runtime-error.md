---
title: 关于 Runtime Error
date: 2018-01-26 14:59:53
tags:
  - C/C++
categories: 
  - 400-编程
  - C/C++
---

自己开发的程序遇到过几次 Runtime Error，用 Visual Studio 调试也完全无法捕获错误，一直以为是运行库有问题，后来发现程序里进行了**错误的类型转换**也可能会引起这个问题。

<!-- more -->
![Runtime Error](/images/runtime-error.jpg )

>Microsoft Visual C++ Runtime Library
>
>Runtime Error!
>
>Program: C:\xxx\xxx.exe
>
>This application has requested the Runtime to terminate it in an unusual way.
>Please contact the application's support team for more information.

