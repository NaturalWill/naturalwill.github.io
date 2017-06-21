title: Ubuntu 16.04 安装PHP、MySQL、Nginx
tags:
  - Linux
  - PHP
categories: []
date: 2017-06-21 10:33:00
---
## 更新 apt 包

	sudo apt update
    
## 安装 MySQL
	
	sudo apt install mysql-server mysql-client
 
## 安装 PHP 7
 
	sudo apt install php-{cli,fpm}

安装PHP插件

	sudo apt install php-{mcrypt,mbstring,curl,gd,mysql,xml}

## 安装 Nginx

	sudo apt install nginx-full