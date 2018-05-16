---
title: Linux 使用技巧
date: 2018-04-16 13:43:07
tags:
categories:
  - 计算机
  - Skills
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