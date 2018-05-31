---
title: 在国内使用 Docker
date: 2018-04-08 14:14:57
tags:
  - Docker
categories:
  - 计算机
  - Notes

---

在国内安装 docker 或是从 Docker Hub 拉取镜像总是遇到网络慢甚至超时等问题。

<!-- more -->

## 安装

### Debian/Ubuntu

如果你过去安装过 docker，先删掉:

    sudo apt-get remove docker docker-engine docker.io

#### 安装依赖

    sudo apt-get install apt-transport-https ca-certificates curl gnupg2 software-properties-common

#### 信任 Docker 的 GPG 公钥

ubuntu

    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

debian

    curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -

#### 添加软件仓库

对于 amd64 架构的计算机，请运行:

    sudo add-apt-repository \
        "deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu \
        $(lsb_release -cs) \
        stable"

如果你是树莓派或其它ARM架构计算机，请运行:

    echo "deb [arch=armhf] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu \
        $(lsb_release -cs) stable" | \
        sudo tee /etc/apt/sources.list.d/docker.list

#### 安装

    sudo apt-get update
    sudo apt-get install docker-ce

### Fedora/CentOS/RHEL

如果你之前安装过 docker，请先删掉

    sudo yum remove docker docker-common docker-selinux docker-engine

#### 安装依赖

    sudo yum install -y yum-utils device-mapper-persistent-data lvm2

#### 添加软件仓库

1. 先下载 repo 文件
    CentOS/RHEL:
    `wget -O /etc/yum.repos.d/docker-ce.repo https://download.docker.com/linux/centos/docker-ce.repo`
    Fedora:
    `wget -O /etc/yum.repos.d/docker-ce.repo https://download.docker.com/linux/fedora/docker-ce.repo`
2. 把软件仓库地址替换为 TUNA:
    `sudo sed -i 's+download.docker.com+mirrors.tuna.tsinghua.edu.cn/docker-ce+' /etc/yum.repos.d/docker-ce.repo`

#### 安装

    sudo yum makecache fast
    sudo yum install docker-ce

## 普通用户使用

将用户加入 docker 用户组

    $ gpasswd -a user docker

## 使用国内的镜像仓库

这是 Docker 官方提供的中国 registry mirror

    registry.docker-cn.com

### 临时使用

pull 时，加上 registry 地址，如：

    $ docker pull registry.docker-cn.com/library/ubuntu

### 永久使用

方法一： 将 `https://registry.docker-cn.com` 加入到 `/etc/docker/daemon.json` 中的 registry-mirrors 数组中。修改后的文件内容应该是这样的。

    {
        "registry-mirrors": ["https://registry.docker-cn.com"]
    }

重启 Docker 服务。

方法二： 启动 Docker daemon 时，带上 --registry-mirror 参数。如。

    $ dockerd --registry-mirror=https://registry.docker-cn.com


## 安装 docker-compose

下载 compose 文件

    sudo curl -L https://github.com/docker/compose/releases/download/1.20.1/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose

添加执行权限

    sudo chmod +x /usr/local/bin/docker-compose