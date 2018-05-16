---
title: "在 Ubuntu 16.04 中安装 Kurento Server"
date: 2018-05-15 15:54:07
tags:
  - Kurento
categories:
  - 计算机
  - Linux
  - 部署

---

## 安装

```sh
REPO="xenial"
echo "deb http://ubuntu.kurento.org $REPO kms6" | sudo tee /etc/apt/sources.list.d/kurento.list
wget http://ubuntu.kurento.org/kurento.gpg.key -O - | sudo apt-key add -
apt update
useradd -U -m kurento
apt install -y kurento-media-server-6.0
systemctl start kurento-media-server-6.0
systemctl enable kurento-media-server-6.0
```

<!-- more -->

## 添加 H.264 支持

安装插件 `openh264-gst-plugins-bad-1.5`

    apt install -y openh264-gst-plugins-bad-1.5

重启 kurento server

    systemctl restart kurento-media-server-6.0

若要只支持 H.264 则需要打开 `/etc/kurento/modules/kurento/SdpEndpoint.conf.json` ，注释掉 videoCodecs 中的 "VP8/90000" 。保存并重启 Kurento Server 。