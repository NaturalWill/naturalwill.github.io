---
title: 代理和镜像设置
date: 2018-02-10 13:52:18
tags:
  - Proxy
  - Notes
categories: 
  - 计算机
  - Notes
---

代理地址格式： `protocol://[user[:pass]@]host[:port]/`

假设代理地址为： <http://127.0.0.1:1080>

<!-- more -->

### Linux

    export http_proxy="http://127.0.0.1:1080"

    export https_proxy="http://127.0.0.1:1080"

<!-- 
    export ftp_proxy="http://127.0.0.1:1080"
-->

### Git

    git config --global https.proxy http://127.0.0.1:1080

    git config --global https.proxy http://127.0.0.1:1080

    git config --global --unset http.proxy

    git config --global --unset https.proxy

### Npm

#### 代理

npm获取配置有 6 种方式，优先级由高到底。

1. 命令行参数。 

        npm install --proxy http://127.0.0.1:1080 [name]

2. 环境变量。 以 `npm_config_` 为前缀的环境变量将会被认为是 npm 的配置属性。

        export npm_config_proxy=http://127.0.0.1:1080

3. 用户配置文件。可以通过 `npm config get userconfig` 查看文件路径。

        npm config set proxy http://127.0.0.1:1080
        npm config set https-proxy http://127.0.0.1:1080

4. 全局配置文件。可以通过 `npm config get globalconfig` 查看文件路径。

        npm config set proxy http://127.0.0.1:1080
        npm config set https-proxy http://127.0.0.1:1080

5. 内置配置文件。安装 npm 的目录下的 `npmrc` 文件。

6. 默认配置。 npm 本身有默认配置参数，如果以上 5 条都没设置，则 npm 会使用默认配置参数。

#### 镜像

* 淘宝npm镜像
        搜索地址：<http://npm.taobao.org/>
        registry地址：<http://registry.npm.taobao.org/>
* cnpmjs镜像
        搜索地址：<http://cnpmjs.org/>
        registry地址：<http://r.cnpmjs.org/>

1. 临时使用

        npm --registry https://registry.npm.taobao.org install [name]

2. 持久使用

        npm config set registry https://registry.npm.taobao.org

        // 配置后可通过下面方式来验证是否成功
        npm config get registry
        // 或
        npm info [name]

3. 通过cnpm使用

        npm install -g cnpm --registry=https://registry.npm.taobao.org

        // 使用
        cnpm install [name]