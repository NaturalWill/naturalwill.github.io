---
title: Linux 使用技巧
date: 2018-04-16 13:43:07
tags:
categories:
  - 400-软件使用
  - Linux
---

## 文件权限

批量修改当前目录及子目录中的文件夹权限为 775 、文件权限为 664

```sh
find . -type f -exec chmod 664 {} \+ -o -type d -exec chmod 775 {} \+
```

## find

find 命令中 `exec` 参数 `+` 结尾与 `;` 结尾有什么区别？

<!-- more -->

> Why is there a difference in output between using
>
>     find . -exec ls '{}' \+
>
> and
>
>     find . -exec ls '{}' \;
>
> This might be best illustrated with an example. Let's say that find turns up these files:
>
>     file1
>     file2
>     file3
>
> Using -exec with a semicolon (find . -exec ls '{}' \;), will execute
>
>     ls file1
>     ls file2
>     ls file3
>
> But if you use a plus sign instead (find . -exec ls '{}' \+), as many filenames as possible are passed as arguments to a single command:
>
>     ls file1 file2 file3
>
> The number of filenames is only limited by the system's maximum command line length. If the command exceeds this length, the command will be called multiple times.

## 磁盘

以下命令可以查看磁盘各分区大小、已用空间等信息：

    df -h

以下命令可以查看foo目录的大小：

    du -sh foo

有时候，硬盘比较满了，我们想找一些目录来清除，可以用下面命令查看当前目录以下搜索文件和子目录大小。找出特别大的，看里面有没有文件可删：

    du -sh *

如果我们插入了一个U盘或移动硬盘，可以用df命令查看它挂载的地方，通常在/mnt或/media下。如果想卸载USB存储设备，可以用umount命令：

    umount path