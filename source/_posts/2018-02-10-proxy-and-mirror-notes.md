---
title: 代理和镜像设置
date: 2018-02-10 13:52:18
tags:
  - proxy
  - mirror
  - npm
  - yarn
categories: 
  - 400-软件使用
  - Notes
---

代理地址格式： `protocol://[user[:pass]@]host[:port]/`

假设代理地址为： <http://127.0.0.1:1080>

<!-- more -->

### Linux

#### 在环境变量中设置代理

```sh
## Set the proxy address of your uni/company/vpn network ## 
export http_proxy=http://127.0.0.1:1080
export https_proxy=http://127.0.0.1:1080

## FTP version ##
export ftp_proxy=http:///127.0.0.1:1080

## http_proxy with username and password 
export http_proxy=http://user:password@your-proxy-ip-address:port/
export https_proxy=https://user:password@your-proxy-ip-address:port/
```

#### 使用配置文件

##### wget

为wget使用代理，可以直接修改`/etc/wgetrc`，也可以在主文件夹下新建`.wgetrc`，并编辑相应内容，一般采用后者。

将`/etc/wgetrc`中与 proxy 有关的几行复制到`~/.wgetrc`，并做如下修改：

```ini
# You can set the default proxies for Wget to use for http, https, and ftp.
# They will override the value in the environment.
https_proxy = http://127.0.0.1:1080
http_proxy = http://127.0.0.1:1080
ftp_proxy = http://127.0.0.1:1080

# If you do not want to use proxy at all, set this to off.
use_proxy = on
```

##### curl

编辑 `~/.curlrc` ：

```ini
proxy="http://127.0.0.1:1080"
```

### Git

    git config --global https.proxy http://127.0.0.1:1080
    git config --global https.proxy http://127.0.0.1:1080

    git config --global http.proxy 'socks5://127.0.0.1:1080'
    git config --global https.proxy 'socks5://127.0.0.1:1080'


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


##### yarn

设置为官方镜像：

        yarn config set registry https://registry.yarnpkg.com

设置为淘宝镜像：

        yarn config set registry http://registry.npm.taobao.org

###### CLI commands comparison

| npm (v5) | Yarn    |
|  ----    | ----    |
| `npm install` | `yarn install` |
| (N/A) | `yarn install --flat` |
| (N/A) | `yarn install --har` |
| `npm install --no-package-lock` | `yarn install --no-lockfile` |
| (N/A) | `yarn install --pure-lockfile` |
| `npm install [package] --save` | `yarn add [package]` |
| `npm install [package] --save-dev` | `yarn add [package] --dev` |
| (N/A) | `yarn add [package] --peer` |
| `npm install [package] --save-optional` | `yarn add [package] --optional` |
| `npm install [package] --save-exact` | `yarn add [package] --exact` |
| (N/A) | `yarn add [package] --tilde` |
| `npm install [package] --global` | `yarn global add [package]` |
| `npm update --global` | `yarn global upgrade` |
| `npm rebuild` | `yarn add --force` |
| `npm uninstall [package]` | `yarn remove [package]` |
| `npm cache clean` | `yarn cache clean [package]` |
| `rm -rf node_modules && npm install` | `yarn upgrade` |
| `npm version major` | `yarn version --major` |
| `npm version minor` | `yarn version --minor` |
| `npm version patch` | `yarn version --patch` |