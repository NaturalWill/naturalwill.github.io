title: Ubuntu 16.04 安装PHP、MySQL、Nginx
tags:
  - Linux
  - PHP
categories: 
  - 计算机
  - Linux
  - 部署
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

配置Nginx, 使之支持PHP, 编辑`/etc/nginx/sites-enabled/default`, 在server段中找到以下一段并解除部分注释，使其如下：


	# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
	#
	location ~ \.php$ {
		include snippets/fastcgi-php.conf;
	
		# With php7.0-cgi alone:
		#fastcgi_pass 127.0.0.1:9000;
		# With php7.0-fpm:
		fastcgi_pass unix:/run/php/php7.0-fpm.sock;
	}