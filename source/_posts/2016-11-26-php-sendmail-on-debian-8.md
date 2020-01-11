layout: blog
title: "Debian 8 下配置 exim4 使 PHP 可以用 sendmail 发送邮件"
date: 2016-11-26 16:23:12
tags: 
  - PHP
  - Linux
  - email
categories: 
  - 计算机
  - Service
  
---

## 简介

e-mail 系统共有三个主要的组成功能。首先是 Mail User Agent (MUA)，这是用户发送和读取邮件的程序。然后是 Mail Transfer Agent (MTA)，用来将邮件从一台计算机传递到另一台。最后是 Mail Delivery Agent (MDA)，用于将收到的邮件投递到用户的收件箱。

这三项功能可以由不同的程序执行，但也能合并到一个或两个程序里。还可以用不同的程序处理不同类型的邮件。

在 Linux 和 Unix 系统上 mutt 是历史悠久的常用 MUA。像其他传统的 Linux 程序一样，是基于纯文本的。它常与作为 MTA 的 exim 或 sendmail、作为 MDA 的 procmail 一起使用。

Debian 8 默认会安装 exim4 和 mutt 软件包。 exim4 组合了 MTA/MDA 功能并相对小巧和灵活。它默认配置为只处理系统本地的 e-mail 。

<!-- more -->

## Sept 1 - 配置 Exim4 MTA

要让系统处理外部 e-mail，需要重新配置 exim4 软件包。

	# dpkg-reconfigure exim4-config
	
> 输入命令之后(作为 root)，您会被问到是否需要将配置文件分成几个小文件。如果您拿不准，就选择默认选项。
>
> 接着您将看到几个常见的邮件方案，请选择一个最近似您需求的那个。
>
> * internet site
> 
> 您的系统被连接到网络上，并且您通过 SMTP 直接收发邮件。在接下来的几页中，程序会询问您一些基本问题，如：您的机器的邮件名称、您接受或转发邮件的域等等。
> 
> * mail sent by smarthost
> 
> 本方案中您的送出邮件转发到另一台机器，称为 “smarthost”，它来负责发送信息到最终目的地。smarthost 一般还用于保存您的计算机接收的邮件，所以您不需要长时间在线。这也意味着您需要使用类似 fetchmail 这样的程序从 smarthost 下载邮件。
> 
> 大多时候 smarthost 是您 ISP 的邮件服务器，这对拨号用户非常适合。它也可以是公司的邮件服务器，或是您自己网络中的另外一台机器。
> 
> * mail sent by smarthost; no local mail
> 
> 该选项基本上与前一种情况相同，只有一点不同，本系统不再架设用于处理本地的 e-mail domain。在本系统上的邮件(比如，给系统管理员的)还是会被处理。
> 
> * local delivery only
> 
> 本选项是系统默认的配置。
> 
> * no configuration at this time
> 
> 除非您真的知道这是在干什么，否则请不要选择这一选项。这会留下一个未配置的邮件系统 — 在您再次配置它之前，您都无法收发任何邮件，并且可能会错过一些系统工具发来的重要信息。
> 
> 如果没有合适的方案，或者需要更精确的设置，您需要在安装完成之后编辑 /etc/exim4 目录下的配置文件。有关 exim4 更多的信息可以在 /usr/share/doc/exim4 下找到；README.Debian.gz 里面有 exim4 配置方面的细节，并说明从哪里找到更多的文档。
> 
> 注意，如果您没有正式的域名，直接发送邮件到互联网，因为接收服务器的反垃圾邮件策略会拒绝接收邮件。这时建议使用 ISP 的邮件服务器。假如您还想直接发送邮件，可能要用另一个邮件地址替换默认生成的那个。如果您使用的是 exim4 作为 MTA，可以添加一个条目到 /etc/email-addresses。

输入命令之后，我们选择 `internet site` ，然后按提示输入域名等相关信息。


## Step 2 - 配置 php.ini

编辑 php.ini ，在其中找到 sendmail_path 选项，如果 sendmail_path 为 sendmail_path = /usr/sbin/sendmail -t -i 即无需修改，若 sendmail_path 为注释状态，则去掉 sendmail_path 前面的“ ; ”并附上 /usr/sbin/sendmail -t -i ，最终会是这样的：

	sendmail_path = /usr/sbin/sendmail -t -i

最后，重启 Apache 或 PHP-FPM 即可。

