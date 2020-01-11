---
title: 为通过yum安装的Nginx添加nginx-rtmp-module
date: 2017-09-25 17:09:40
tags:
  - Linux
  - Nginx
  - 流媒体
  - RTMP
categories: 
  - 计算机
  - Service
---

Nginx 一般都是编译安装，不过它本身也提供了通过yum安装的方式，比如在CentOS 7中需要先安装 Yum 源：

```sh
rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

之后就是常规的 `yum -y install nginx` 。

通过这一方式安装的Nginx已经可以通过系统服务的方式进行启动，相当便捷，但是很多有趣的第三方插件并没有能够加入，比如header-more-nginx-module，Nginx目前看来不像Tengine是可以后期动态添加模块的，所以解决的方案出了编译安装似乎没有其他的方式了。

不过Nginx编译只生成一个二进制文件，那么，如果获取yum安装的Nginx编译参数，之后使用同一版本的源代码进行编译，之后替换生成文件就可以了。


<!-- more -->

## 安装 nginx

	yum -y install nginx

	
## 查看原来的 nginx 编译参数

	nginx -V
	
输出如下：
	
	nginx version: nginx/1.12.2
	built by gcc 4.8.5 20150623 (Red Hat 4.8.5-16) (GCC)
	built with OpenSSL 1.0.2k-fips  26 Jan 2017
	TLS SNI support enabled
	configure arguments: --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi --http-scgi-temp-path=/var/lib/nginx/tmp/scgi --pid-path=/run/nginx.pid --lock-path=/run/lock/subsys/nginx --user=nginx --group=nginx --with-file-aio --with-ipv6 --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_addition_module --with-http_xslt_module=dynamic --with-http_image_filter_module=dynamic --with-http_geoip_module=dynamic --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_slice_module --with-http_stub_status_module --with-http_perl_module=dynamic --with-mail=dynamic --with-mail_ssl_module --with-pcre --with-pcre-jit --with-stream=dynamic --with-stream_ssl_module --with-google_perftools_module --with-debug --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -m64 -mtune=generic' --with-ld-opt='-Wl,-z,relro -specs=/usr/lib/rpm/redhat/redhat-hardened-ld -Wl,-E' --add-module=../nginx-rtmp-module

	
## 获取对应的 Nginx 源码

	wget https://nginx.org/download/nginx-1.12.2.tar.gz

获取nginx-rtmp-module源码：
	
	git clone https://github.com/arut/nginx-rtmp-module.git
	cd nginx-rtmp-module
	git checkout $(git describe --abbrev=0 --tags)
	
## 安装依赖
	
	yum -y install {GeoIP,gd,gperftools,libxml2,libxslt,openssl,pcre,perl,zlib}-devel perl-ExtUtils-Embed redhat-rpm-config

## 编译安装
	
在获得的编译参数中加入 nginx-rtmp-module 源码的路径，如： `--add-module=../nginx-rtmp-module` 。然后执行 configure
	
	./configure --prefix=/usr/share/nginx \
		--sbin-path=/usr/sbin/nginx \
		--modules-path=/usr/lib64/nginx/modules \
		--conf-path=/etc/nginx/nginx.conf \
		--error-log-path=/var/log/nginx/error.log \
		--http-log-path=/var/log/nginx/access.log \
		--http-client-body-temp-path=/var/lib/nginx/tmp/client_body \
		--http-proxy-temp-path=/var/lib/nginx/tmp/proxy \
		--http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi \
		--http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi \
		--http-scgi-temp-path=/var/lib/nginx/tmp/scgi \
		--pid-path=/run/nginx.pid \
		--lock-path=/run/lock/subsys/nginx \
		--user=nginx \
		--group=nginx \
		--with-file-aio \
		--with-ipv6 \
		--with-http_ssl_module \
		--with-http_v2_module \
		--with-http_realip_module \
		--with-http_addition_module \
		--with-http_xslt_module=dynamic \
		--with-http_image_filter_module=dynamic \
		--with-http_geoip_module=dynamic \
		--with-http_sub_module \
		--with-http_dav_module \
		--with-http_flv_module \
		--with-http_mp4_module \
		--with-http_gunzip_module \
		--with-http_gzip_static_module \
		--with-http_random_index_module \
		--with-http_secure_link_module \
		--with-http_degradation_module \
		--with-http_slice_module \
		--with-http_stub_status_module \
		--with-http_perl_module=dynamic \
		--with-mail=dynamic \
		--with-mail_ssl_module \
		--with-pcre \
		--with-pcre-jit \
		--with-stream=dynamic \
		--with-stream_ssl_module \
		--with-google_perftools_module \
		--with-debug \
		--with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -m64 -mtune=generic' \
		--with-ld-opt='-Wl,-z,relro -specs=/usr/lib/rpm/redhat/redhat-hardened-ld -Wl,-E' \
		--add-module=../nginx-rtmp-module
	 
停止运行现在的Nginx，然后执行编译安装：

	systemctl stop nginx

	make
	make install


## 禁止 yum 更新 nginx

修改 yum.conf :

	nano /etc/yum.conf

在文件中加入下面配置顶：

	exclude=nginx*

修改后的 yum.conf 看起来是这样的：

	[main]
	cachedir=/var/cache/yum/$basearch/$releasever
	keepcache=0
	debuglevel=2
	logfile=/var/log/yum.log
	exactarch=1
	obsoletes=1
	gpgcheck=1
	plugins=1
	installonly_limit=5
	bugtracker_url=http://bugs.centos.org/set_project.php?project_id=19&ref=http://bugs.centos.org/bug_report_page.php?category=yum
	distroverpkg=centos-release

	#  This is the default, if you make this bigger yum won't see if the metadata
	# is newer on the remote and so you'll "gain" the bandwidth of not having to
	# download the new metadata and "pay" for it by yum not having correct
	# information.
	#  It is esp. important, to have correct metadata, for distributions like
	# Fedora which don't keep old packages around. If you don't like this checking
	# interupting your command line usage, it's much better to have something
	# manually check the metadata once an hour (yum-updatesd will do this).
	# metadata_expire=90m

	# PUT YOUR REPOS HERE OR IN separate files named file.repo
	# in /etc/yum.repos.d

	exlude=nginx*


这样，使用 yum update 命令升级系统的时候就不会再升级 nginx 相关的软件包了。


参考链接：

[ 为yum安装的Nginx添加模块 ]( https://anyof.me/articles/236 )

