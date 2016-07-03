---
layout: post
title: ArchLinux下修改wine的默认浏览器
tags: [Linux, Archlinux, wine]
date: 2016/2/24
---

wine 1.9 已经可以运行 QQ 7.1 了，体验已经算是很不错了，唯一感觉美中不足的就是在QQ上点开邮箱时，用的wine iexplore不能正常打开QQ邮箱，Google了一下，发现网上只有通过修改注册表的方式修改wine的默认浏览器，然而我按网上的方法来发现根本没有那些注册表项，于是只好自己想办法了。

--------

首先，打开gnome自带的系统监视器，找到当前运行的iexplore的位置

`/home/username/.wine/drive_c/Program Files (x86)/Internet Explorer/iexplore.exe`

然后，把ie替换为我们想要用的浏览器。

具体做法：

1. 为以防万一，先备份ie浏览器

        $ cd ～/.wine/drive_c/Program\ Files\ \(x86\)/Internet\ Explorer
        $ mv iexplore.exe iexplore.exe.backup

2. 找到自己正在用的浏览器的位置，创建软连接，比如我用的是Archlinux AUR里的google-chrome

        $ ln -s $(which google-chrome-stable) iexplore.exe

完成！当我们再次点击QQ邮箱的图标，打开邮箱的浏览器已经变成Google-chrome了～～
（如果你使用的是其他的浏览器，只需要把上面的`google-chrome-stable`改为你在终端中打开浏览器的命令就可以。）

PS: 附上 google-chrome-stable 文件内容：

        $ cat $(which google-chrome-stable)

        #!/bin/bash

        # Allow users to override command-line options
        if [[ -f ~/.config/chrome-flags.conf ]]; then
                CHROME_USER_FLAGS="$(cat ~/.config/chrome-flags.conf)"
        fi

        # Launch
        exec /opt/google/chrome/google-chrome $CHROME_USER_FLAGS "$@"%


