---
title: Ubuntu 16.04 禁用客人会话
date: 2018-01-22 16:39:13
tags:
  - Ubuntu
  - Notes
categories: 
  - 计算机
  - Linux
  - Notes
---

禁用Ubuntu 16.04 的访客模式，只需要修改配置文件 `/etc/lightdm/lightdm.conf.d/50-guest-wrapper.conf` 。

    sudo nano /etc/lightdm/lightdm.conf.d/50-guest-wrapper.conf

在文件中添加以下内容即可。

	[Seat:*]
	allow-guest=false
