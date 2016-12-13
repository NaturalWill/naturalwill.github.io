layout: blog
title: "【怪事】在 Arch on Windows Subsystem 里运行 hexo g 生成的部分博客的日期早了一天"
date: 2016-12-13 14:12:19
tags: 
  - Hexo
  - Node.js
  - WSL
  - issue

---

## 缘起

今天突然发现一件怪事，我博客的一篇文章的地址变了，原本是

	https://naturalwill.github.io/2016/02/24/Modify-wine-default-webbrowser-in-archlinux/

却突然变成了

	https://naturalwill.github.io/2016/02/23/Modify-wine-default-webbrowser-in-archlinux/

日期部分早了一天。

回想最近做了什么，唯一与这博客有关的事情就是我在 archlinux as the WSL (Windows Subsystem for Linux) 里生成了一次博客。

打开 alwsl 重新生成一次，发现，果然如此。

这是我的博客文件列表：

	jier@DESKTOP-BI1IICL ~/repo/hexo-blog (git)-[master] % ls source/_posts
	2015-12-16-Start-VirtualBox-Machine-With-Login-In-Windows.md
	2015-12-23-PHP-Convert-between-JSON-and-XML.md
	2015-12-30-Simple-Healthcare-Consulting-System-API.md
	2016-02-24-Modify-wine-default-webbrowser-in-archlinux.md
	2016-07-03-compress-and-extract-files-on-linux.md
	2016-07-07-Secure-LNMP-on-Ubuntu-14-04.md
	2016-08-24-Create-Facade-on-Laravel-5-2.md
	2016-11-26-JavaScript-the-world-s-most-misunderstood-programming-language.md
	2016-11-26-php-sendmail-on-debian-8.md

在 alwsl 里运行 `hexo g` :

	jier@DESKTOP-BI1IICL ~/repo/hexo-blog (git)-[master] % hexo g
	INFO  Start processing
	INFO  Files loaded in 2.68 s
	INFO  Generated: index.html
	INFO  Generated: 2016/11/26/php-sendmail-on-debian-8/index.html
	INFO  Generated: 2015/12/15/Start-VirtualBox-Machine-With-Login-In-Windows/index.html
	INFO  Generated: 2016/07/06/Secure-LNMP-on-Ubuntu-14-04/index.html
	INFO  Generated: 2016/11/26/JavaScript-the-world-s-most-misunderstood-programming-language/index.html
	INFO  Generated: 2016/02/23/Modify-wine-default-webbrowser-in-archlinux/index.html
	INFO  Generated: 2016/08/23/Create-Facade-on-Laravel-5-2/index.html
	INFO  Generated: 2016/07/02/compress-and-extract-files-on-linux/index.html
	INFO  Generated: 2015/12/29/Simple-Healthcare-Consulting-System-API/index.html
	INFO  Generated: 2015/12/22/PHP-Convert-between-JSON-and-XML/index.html
	INFO  10 files generated in 27 s
	hexo g  9.19s user 30.41s system 56% cpu 1:10.66 total

在 ArchLinux 里运行 `hexo g` :

	jier@zarch ~/hexo-blog (git)-[master] % hexo g
	INFO  Start processing
	INFO  Files loaded in 1.55 s
	INFO  Generated: 2016/11/26/php-sendmail-on-debian-8/index.html
	INFO  Generated: 2016/11/26/JavaScript-the-world-s-most-misunderstood-programming-language/index.html
	INFO  Generated: 2016/08/24/Create-Facade-on-Laravel-5-2/index.html
	INFO  Generated: 2016/07/07/Secure-LNMP-on-Ubuntu-14-04/index.html
	INFO  Generated: 2016/07/03/compress-and-extract-files-on-linux/index.html
	INFO  Generated: 2016/02/24/Modify-wine-default-webbrowser-in-archlinux/index.html
	INFO  Generated: 2015/12/30/Simple-Healthcare-Consulting-System-API/index.html
	INFO  Generated: 2015/12/23/PHP-Convert-between-JSON-and-XML/index.html
	INFO  Generated: 2015/12/16/Start-VirtualBox-Machine-With-Login-In-Windows/index.html
	INFO  Generated: index.html
	INFO  10 files generated in 2.34 s

## 解决方案

寻找中。。。