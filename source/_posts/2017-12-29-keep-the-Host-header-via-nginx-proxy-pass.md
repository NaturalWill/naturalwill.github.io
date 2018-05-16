---
title: 使用 Nginx 转发时保留原始域名
date: 2017-12-29 17:03:56
tags:
  - Nginx
categories: 
  - 计算机
  - Linux
  - Notes
---

如果直接通过设置 proxy_pass 转发，会发现原来的域名会丢失，取代出现的是目标地址的 IP 。

    server {
        listen 80;

        location / {
        proxy_pass http://127.0.0.1:4356;
        }
    }

解决方法是设置 proxy_set_header . 具体如下：

    server {
        listen 80;

        # this is the key !!!
        proxy_set_header Host $host:$server_port;

        location / {
            proxy_pass http://127.0.0.1:4356;
        }
    }