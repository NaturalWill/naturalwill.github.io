---
layout: post
title: Ubuntu 14.04 上部署LNMP，并开启HTTPS
tags: [Linux, Ubuntu, Nginx, MySQL, PHP, HTTPS]
date: 2016/07/07
---


本教程将会涉及以下工具：

* Ubuntu 14.04
* MySQL
* PHP
* Nginx

## 安装

更新软件源

	sudo apt-get update

安装必要组件

	sudo apt-get install mysql-server   # 安装 MySQL
	
	sudo apt-get install nginx-full     # 安装 Nginx

	sudo apt-get install php5-fpm       # 安装 php5
	sudo apt-get install php5-cli       # 安装 php5 命令行工具
	sudo apt-get install php5-mcrypt php5-mysql php5-curl    # 安装常用 php5 扩展

安装 PHP 依赖管理工具 composer (可选)

	sudo apt-get install git     # 安装 git

	# 安装 composer
	cd ~
	curl -sS https://getcomposer.org/installer | php
	sudo mv composer.phar /usr/local/bin/composer


## 配置

### PHP 配置

打开配置文件：

	sudo nano /etc/php5/fpm/php.ini

找到 cgi.fix_pathinfo 修改为 0 ，如下：

	cgi.fix_pathinfo=0

保存并退出，因为这是一个可能的安全漏洞，详情可以看[鸟哥的文章](http://www.laruence.com/2010/05/20/1495.html)！

使用 php5enmod 启用 MCrypt 扩展，并重启 php5-fpm 服务：

	sudo php5enmod mcrypt
	sudo service php5-fpm restart


### Nginx 配置

创建网站目录 ：

	sudo mkdir -p /var/www/default

打开 nginx 默认配置文件：

	sudo nano /etc/nginx/sites-available/default

默认配置如下：

	# You may add here your
	# server {
	#       ...
	# }
	# statements for each of your virtual hosts to this file

	##
	# You should look at the following URL's in order to grasp a solid understanding
	# of Nginx configuration files in order to fully unleash the power of Nginx.
	# http://wiki.nginx.org/Pitfalls
	# http://wiki.nginx.org/QuickStart
	# http://wiki.nginx.org/Configuration
	#
	# Generally, you will want to move this file somewhere, and start with a clean
	# file but keep this around for reference. Or just disable in sites-enabled.
	#
	# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
	##

	server {
			listen 80 default_server;
			listen [::]:80 default_server ipv6only=on;

			root /usr/share/nginx/html;
			index index.html index.htm;

			# Make site accessible from http://localhost/
			server_name localhost;

			location / {
					# First attempt to serve request as file, then
					# as directory, then fall back to displaying a 404.
					try_files $uri $uri/ =404;
					# Uncomment to enable naxsi on this location
					# include /etc/nginx/naxsi.rules
			}

			# Only for nginx-naxsi used with nginx-naxsi-ui : process denied requests
			#location /RequestDenied {
			#       proxy_pass http://127.0.0.1:8080;
			#}

			#error_page 404 /404.html;

			# redirect server error pages to the static page /50x.html
			#
			#error_page 500 502 503 504 /50x.html;
			#location = /50x.html {
			#       root /usr/share/nginx/html;
			#}

			# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
			#
			#location ~ \.php$ {
			#       fastcgi_split_path_info ^(.+\.php)(/.+)$;
			#       # NOTE: You should have "cgi.fix_pathinfo = 0;" in php.ini
			#
			#       # With php5-cgi alone:
			#       fastcgi_pass 127.0.0.1:9000;
			#       # With php5-fpm:
			#       fastcgi_pass unix:/var/run/php5-fpm.sock;
			#       fastcgi_index index.php;
			#       include fastcgi_params;
			#}

			# deny access to .htaccess files, if Apache's document root
			# concurs with nginx's one
			#
			#location ~ /\.ht {
			#       deny all;
			#}
	}


	# another virtual host using mix of IP-, name-, and port-based configuration
	#
	#server {
	#       listen 8000;
	#       listen somename:8080;
	#       server_name somename alias another.alias;
	#       root html;
	#       index index.html index.htm;
	#
	#       location / {
	#               try_files $uri $uri/ =404;
	#       }
	#}


	# HTTPS server
	#
	#server {
	#       listen 443;
	#       server_name localhost;
	#
	#       root html;
	#       index index.html index.htm;
	#
	#       ssl on;
	#       ssl_certificate cert.pem;
	#       ssl_certificate_key cert.key;
	#
	#       ssl_session_timeout 5m;
	#
	#       ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2;
	#       ssl_ciphers "HIGH:!aNULL:!MD5 or HIGH:!aNULL:!MD5:!3DES";
	#       ssl_prefer_server_ciphers on;
	#
	#       location / {
	#               try_files $uri $uri/ =404;
	#       }
	#}


修改如下：

	server {
		# 监听 80 端口
		listen 80 default_server;
		listen [::]:80 default_server ipv6only=on;
		
		# 监听 443 端口
		listen 443 ssl;
		listen [::]:443 ssl ipv6only=on;
		
		 # 服务器名称，server_domain_or_IP 请替换为自己设置的名称或者 IP 地址
		server_name server_domain_or_IP;
		
		# 设定网站根目录
		root /var/www/default;
		# 网站默认首页
		index index.php index.html index.htm;

		# 开启 SSL 并设置证书路径
		ssl on;
		ssl_certificate  /path/to/your/domain/cert/cert.crt;
		ssl_certificate_key /path/to/your/domain/cert/cert.key;

		# SSL 配置
		ssl_session_timeout 5m;
		ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2;
		ssl_ciphers "HIGH:!aNULL:!MD5 or HIGH:!aNULL:!MD5:!3DES";
		ssl_prefer_server_ciphers on;
		
		# 强制用 HTTPS 访问，如不需要可以注释掉
		add_header Strict-Transport-Security max-age=63072000;
		add_header X-Frame-Options DENY;
		add_header X-Content-Type-Options nosniff;

		location / {
			try_files $uri $uri/ =404;
			
			# Laravel 转发规则，若使用 Laravel 框架，则修改为此配置
			#try_files $uri $uri/ /index.php?$query_string;  
		}
		

		# PHP 支持
		location ~ \.php$ {
			try_files $uri /index.php =404;
			fastcgi_split_path_info ^(.+\.php)(/.+)$;
			fastcgi_pass unix:/var/run/php5-fpm.sock;
			fastcgi_index index.php;
			fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
			include fastcgi_params;
		}
	}

重启 nginx 服务：

	sudo service nginx restart

## 运行

在浏览器打开上面设置的 `server_domain_or_IP` 即可。