---
layout: post
title: 两步实现开机登陆自动后台运行VirtualBox虚拟机
tags: 
  - VirtualBox
  - Windows
categories: 
  - 400-软件使用
  - Windows
date: 2015/12/16
---


目的：开机后自动后台运行虚拟机，只有进程，不显示GUI界面，关机自动保存虚拟机状态，以便下次开机恢复。

### Step 1

#### 启动虚拟机脚本: startvm.bat

```batch
@echo off
"D:\Program Files\Oracle\VirtualBox\VBoxManage" startvm Arch_x64 --type headless
```

#### 休眠虚拟机脚本: savestate.bat
```batch
@echo off
"D:\Program Files\Oracle\VirtualBox\VBoxManage" controlvm Arch_x64 savestate
```

**注意**:

* `"D:\Program Files\Oracle\VirtualBox"`是VirtualBox的安装目录，如果你装在其他地方则需要改为对应的位置。
* `Arch_x64`为虚拟机名字

### Step 2

快捷键 `<Win> + R` 打开运行窗口，输入`gpedit.msc`打开组策略，在`用户配置`->`Windows设置`->`脚本（登录/注销）`里，分别添加上面2个脚本。

------

**恭喜你！你已经实现了开机后自动后台运行虚拟机**。

<!-- more -->

------
### 附：VBoxManage命令部分选项解析

	C:\>"D:\Program Files\Oracle\VirtualBox\VBoxManage"
	Oracle VM VirtualBox Command Line Management Interface Version 5.0.10
	(C) 2005-2015 Oracle Corporation
	All rights reserved.

	Usage:

	  VBoxManage [<general option>] <command>


	General Options:

	  [-v|--version]            print version number and exit
	  [-q|--nologo]             suppress the logo
	  [--settingspw <pw>]       provide the settings password
	  [--settingspwfile <file>] provide a file containing the settings password


	Commands:
	
	  list [--long|-l]          vms|runningvms|ostypes|hostdvds|hostfloppies|
								intnets|bridgedifs|hostonlyifs|natnets|dhcpservers|
								hostinfo|hostcpuids|hddbackends|hdds|dvds|floppies|
								usbhost|usbfilters|systemproperties|extpacks|
								groups|webcams|screenshotformats
								
	...
	
	  startvm                   <uuid|vmname>...
								[--type gui|sdl|headless|separate]

	  controlvm                 <uuid|vmname>
								pause|resume|reset|poweroff|savestate|
								
	...
	
	
* 如果要使用uuid，可以通过 `VBoxManage list vms`查看
* startvm命令中type选项解析
	* gui -- 图形化界面，VirtualBox的默认启动方式
	* sdl -- 图形化界面，但是少掉了部分功能，比如没有菜单等，一般用于调试过程
	* headless -- 后台运行，并且默认开启vrdp服务，可以通过远程桌面工具来访问
	* separate -- 分离式启动，在VirtualBox 5.0版本新增的启动方式：在后台启动虚拟机，分离的前端进程可以关闭，而虚拟机会继续运行
