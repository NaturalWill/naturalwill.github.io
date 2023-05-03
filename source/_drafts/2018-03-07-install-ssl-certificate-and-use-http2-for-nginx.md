---
title: 为 Nginx 安装 Let's Encrypt 证书并启用 Http2
date: 2018-03-07 10:44:59
tags:
---

Let's Encrypt 这个免费、自动化、开放的证书签发服务

https://www.ssllabs.com/ssltest/analyze.html?d=blog.naturalwill.me&latest

## 准备

把要申请证书的域名都解析到了相应的服务器上，且能直接通过域名访问。

## 安装 CertBot

我的运行环境：
- Ubuntu 16.04
- Nginx/1.10.3 （>=1.9.3）

安装 certbot

    $ sudo apt-get update
    $ sudo apt-get install software-properties-common
    $ sudo add-apt-repository ppa:certbot/certbot
    $ sudo apt-get update
    $ sudo apt-get install python-certbot-nginx 

    