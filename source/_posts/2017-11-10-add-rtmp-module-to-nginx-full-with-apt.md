---
title: 通过 Apt 为 Nginx 添加 RTMP 直播模块
date: 2017-11-10 16:07:07
tags:
  - Linux
  - Nginx
  - RTMP
  - 流媒体
categories:
  - 计算机
  - Linux
  - 部署
---

通过 apt 安装的 Nginx 可以通过系统服务的方式进行启动，相当便捷，但 Nginx 目前还不可以后期动态添加模块的，所以只能通过编译安装了。

## 准备

更新，及安装必要工具。

    sudo apt-get update
    sudo apt-get install -y dpkg-dev

创建一个干净的目录，用于后面的编译工作。

    mkdir build && cd build

获取 nginx 源码。执行以下代码后，会在当前目录将 nginx 的源码下载至 _nginx-*_ , * 是版本号。

    apt-get source nginx

获取 nginx-rtmp-module 源码。通过 git 获取，或者[直接下载]( https://github.com/arut/nginx-rtmp-module/archive/master.zip )。

    git clone https://github.com/arut/nginx-rtmp-module.git
    cd nginx-rtmp-module
    git checkout $(git describe --abbrev=0 --tags)

<!-- more -->

## 配置

进入  _nginx-*_ 目录，修改 `debian/rules`, 找到 `full_configure_flags` 或 `config.status.full` 这一段，在最后增加一行 `--add-module=*`，其中 = 后是 nginx-rtmp-module 的路径，比如我的 nginx-rtmp-module 跟 nginx 一样都在 build 目录下，是这样的。

```
--add-module=$(CURDIR)/../nginx-rtmp-module
```

修改后的段落像是这样的。

```
full_configure_flags := \
            $(common_configure_flags) \
            --with-http_addition_module \
            --with-http_dav_module \
            --with-http_geoip_module \
            --with-http_gunzip_module \
            --with-http_gzip_static_module \
            --with-http_image_filter_module \
                        --with-http_v2_module \
            --with-http_sub_module \
            --with-http_xslt_module \
            --with-stream \
            --with-stream_ssl_module \
            --with-mail \
            --with-mail_ssl_module \
            --with-threads \
            --add-module=$(MODULESDIR)/nginx-auth-pam \
            --add-module=$(MODULESDIR)/nginx-dav-ext-module \
            --add-module=$(MODULESDIR)/nginx-echo \
            --add-module=$(MODULESDIR)/nginx-upstream-fair \
            --add-module=$(MODULESDIR)/ngx_http_substitutions_filter_module \
            --add-module=$(CURDIR)/../nginx-rtmp-module
```

或是这样的。

```
config.status.full: config.env.full
        cd $(BUILDDIR_full) && ./configure  \
            $(common_configure_flags) \
            --with-http_addition_module \
            --with-http_dav_module \
            --with-http_geoip_module \
            --with-http_gzip_static_module \
            --with-http_image_filter_module \
            --with-http_spdy_module \
            --with-http_sub_module \
            --with-http_xslt_module \
            --with-mail \
            --with-mail_ssl_module \
            --add-module=$(MODULESDIR)/nginx-auth-pam \
            --add-module=$(MODULESDIR)/nginx-dav-ext-module \
            --add-module=$(MODULESDIR)/nginx-echo \
            --add-module=$(MODULESDIR)/nginx-upstream-fair \
            --add-module=$(MODULESDIR)/ngx_http_substitutions_filter_module \
            --add-module=$(CURDIR)/../nginx-rtmp-module \
            >$@
        touch $@
```

## 安装依赖

    sudo apt-get build-dep nginx

## 构建

    dpkg-buildpackage -b

之后，会在 nginx 的上级目录，也就是 build 目录下，生成一堆 deb 软件包。

## 安装

接下来就可以直接安装了。

    sudo dpkg -i nginx-common_*.deb nginx-full_*.deb


## 后期工作

在运行 `apt upgrade` 的时候，忽略 nginx-common 与 nginx-full 。

    sudo apt-mark hold nginx-common nginx-full
