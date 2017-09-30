---
layout: post
title: TeamTalk server 编译与部署笔记
tags:
  - TeamTalk
  - C++
categories: []
date: 2017-03-29 09:53:00
---
记录 TeamTalk Server 的部署过程，仅供大家参考。


为了简化问题，全程采用 root 操作。


## 编译前准备

### 环境


    操作系统: Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-62-generic x86_64)
    域名： teamtalk.naturalwill.me
    IP: 192.168.1.70

<!-- more -->
    
更新操作系统:

    apt-get update && apt-get upgrade

该命令会执行更新，会消耗一段时间，国内用户，建议使用科大源或者163，搜狐等都可以，这会为大家节省很多时间，具体使用方法，可以见相关的页面:

    清华源帮助：
        https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/

    163源帮助:
        http://mirrors.163.com/.help/ubuntu.html

    搜狐源帮助:
        http://mirrors.sohu.com/help/ubuntu.html

    科大源帮助:
        https://lug.ustc.edu.cn/wiki/mirrors/help/ubuntu




### 安装 MySQL 、 Nginx 、 Redis

    aptitude install nginx-full -y 
    aptitude install redis-server -y 
    aptitude install mysql-server -y
    
    

### 安装 PHP 及 zend-loader
    
安装 PHP 及需要用到的插件
    
```sh
add-apt-repository ppa:ondrej/php
apt-get install -y software-properties-common
apt-get update
apt-get install -y --allow-unauthenticated php5.6-{fpm,cli,dev,gd,mcrypt,mysqli,curl,mbstring,exif}
```

配置 PHP ：

    sed -i 's/post_max_size = 8M/post_max_size = 50M/g' /etc/php/5.6/fpm/php.ini
    sed -i 's/upload_max_filesize = 2M/upload_max_filesize = 50M/g' /etc/php/5.6/fpm/php.ini
    sed -i 's/;date.timezone =/date.timezone = PRC/g' /etc/php/5.6/fpm/php.ini
    sed -i 's/short_open_tag = Off/short_open_tag = On/g' /etc/php/5.6/fpm/php.ini
    sed -i 's/; cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/g' /etc/php/5.6/fpm/php.ini
    sed -i 's/; cgi.fix_pathinfo=0/cgi.fix_pathinfo=0/g' /etc/php/5.6/fpm/php.ini
    sed -i 's/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/g' /etc/php/5.6/fpm/php.ini
    sed -i 's/max_execution_time = 30/max_execution_time = 300/g' /etc/php/5.6/fpm/php.ini
    sed -i 's/register_long_arrays = On/;register_long_arrays = On/g' /etc/php/5.6/fpm/php.ini
    sed -i 's/magic_quotes_gpc = On/;magic_quotes_gpc = On/g' /etc/php/5.6/fpm/php.ini
    sed -i 's/disable_functions =.*/disable_functions = passthru,exec,system,chroot,scandir,chgrp,chown,shell_exec,proc_open,proc_get_status,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,stream_socket_server/g' /etc/php/5.6/fpm/php.ini


安装对应版本的 [ Zend Guard Loader ]( http://www.zend.com/en/products/loader/downloads#Linux ) 


```sh
wget http://downloads.zend.com/guard/7.0.0/zend-loader-php5.6-linux-x86_64_update1.tar.gz
mkdir -p /usr/local/zend/ #1489462850:0
tar -xzvf zend-loader-php5.6-linux-x86_64_update1.tar.gz
cp zend-loader-php5.6-linux-x86_64/*.so /usr/local/zend/
```

修改 PHP 配置文件 `/etc/php/5.6/fpm/php.ini` ，在文件末尾添加：

    ;eaccelerator

    ;ionCube

    [Zend Optimizer]
    zend_extension=/usr/local/zend/ZendGuardLoader.so
    zend_extension=/usr/local/zend/opcache.so
    zend_loader.enable=1
    zend_loader.disable_licensing=0
    zend_loader.obfuscation_level_support=3
    zend_loader.license_path=
	


安装 Termcap:

    wget https://ftp.gnu.org/gnu/termcap/termcap-1.3.1.tar.gz
    tar -xzvf termcap-1.3.1.tar.gz
    cd termcap-1.3.1
    ./configure --prefix=/usr
    make -j 2 && make install

## 编译 Teamtalk

下载并解压：

    wget https://github.com/meili/TeamTalk/archive/master.zip -O TeamTalk-master.zip
    unzip TeamTalk-master.zip
    mv TeamTalk-master ~/TeamTalk
    
### 编译 google protobuf:

    cd ~/TeamTalk/server/src/protobuf/
    tar -xzf protobuf-2.6.1.tar.gz
    cd protobuf-2.6.1/
    ./configure --prefix=/usr/local/protobuf
    make -j 2 && make install
    
拷贝pb相关文件

    mkdir -p ~/TeamTalk/server/src/base/pb/lib/linux/
    cp /usr/local/protobuf/lib/libprotobuf-lite.a ~/TeamTalk/server/src/base/pb/lib/linux/
    cp -r /usr/local/protobuf/include/* ~/TeamTalk/server/src/base/pb/
    
生成pb协议

    cd ~/TeamTalk/pb
    export PATH=$PATH:/usr/local/protobuf/bin
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/protobuf/lib
    bash create.sh
    
将相关文件拷贝到server 目录下：

    bash sync.sh

### 安装 log4cxx:

	apt -y install liblog4cxx-dev liblog4cxx10-dev

	cd ~/TeamTalk/server/src
	rm -rf slog/include
	cp -rf /usr/include/log4cxx slog/include
	
	mkdir -p slog/lib/
	cp -f $(dpkg -L liblog4cxx-dev | grep \\.so$)* slog/lib/

### 安装其他依赖：

    apt -y install libuu-dev libhiredis-dev protobuf-compiler cmake make g++ git libprotobuf-dev libcurl4-openssl-dev openssl

    cd ~/TeamTalk/server/src
    bash ./make_hiredis.sh
    
    aptitude install -y mysql-client libmysqlclient-dev
    ln -s /usr/lib/x86_64-linux-gnu/libmysqlclient.so /usr/lib/x86_64-linux-gnu/libmysqlclient_r.so

### 编译

    cd ~/TeamTalk/server/src
    bash build_ubuntu.sh version 1.0.0
    

## 安装


    cd ~/TeamTalk/server
    tar -xzf im-server-1.0.0.tar.gz
    mv im-server-1.0.0 /usr/local/teamtalk
	cd /usr/local/teamtalk && sh sync_lib_for_zip.sh
    ln -s /usr/local/teamtalk/daeml /usr/local/bin/daeml

    
## 配置



### MySQL

登陆mysql:

    mysql -uroot -p
    
    
创建TeamTalk数据库:

    create database teamtalk
    
见到如下:

    mysql> create database teamtalk;
    Query OK, 1 row affected (0.00 sec)

创建成功。
 
创建teamtalk用户并给teamtalk用户授权teamtalk的操作:

    grant select,insert,update,delete on teamtalk.* to 'teamtalk'@'%' identified by '12345';
    flush privileges;

导入数据库.

    use teamtalk;
    source ~/TeamTalk/auto_setup/mariadb/conf/ttopen.sql;
    show tables;
    
    
    
### PHP 程序配置

执行如下命令:

    mkdir -p /var/www/teamtalk
    cp -r ~/TeamTalk/php/* /var/www/teamtalk/

修改config.php:

	cd /var/www/teamtalk/
	vim application/config/config.php

修改第18-19行:

    $config['msfs_url'] = 'http://teamtalk.naturalwill.me:8700/';
    $config['http_url'] = 'http://teamtalk.naturalwill.me:8400';

修改database.php

    vim application/config/database.php

修改52-54行:

    $db['default']['hostname'] = '127.0.0.1';
    $db['default']['username'] = 'tamtalk';
    $db['default']['password'] = '12345';
    $db['default']['database'] = 'teamtalk';
    
    
### Nginx

创建 Nginx 配置：

    vim /etc/nginx/sites-available/teamtalk.conf

修改如下：
    
    server {
        listen 80 default_server;
        listen [::]:80 default_server;

        server_name teamtalk.naturalwill.me;
        index index.html index.htm index.php default.html default.htm default.php;
        root /var/www/teamtalk;


        location ~ \.php($|/) {
            # fastcgi_pass   127.0.0.1:9000;
            # fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
            fastcgi_pass unix:/var/run/php/php5.6-fpm.sock;
            fastcgi_index  index.php;
            fastcgi_split_path_info ^(.+\.php)(.*)$;
            fastcgi_param   PATH_INFO $fastcgi_path_info;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }

        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
                {
                        expires      30d;
                }

        location ~ .*\.(js|css)?$
                {
                        expires      12h;
                }
        if (!-e $request_filename) {
            rewrite ^/(.*)$ /index.php/$1 last;
            break;
        }
    }

激活配置：

    ln -s /etc/nginx/sites-available/teamtalk.conf /etc/nginx/sites-enabled/teamtalk.conf 
    systemctl reload nginx
    

### TeamTalk 配置

必须修改的配置：

1. db_proxy_server 中的数据库配置
2. msg_server 中的 DBServer 需要有 2 个
3. msfs 中的 BaseDir

配置文件说明:

#### db_proxy_server

    ListenIP=127.0.0.1
    ListenPort=10600
    ThreadNum=48        # double the number of CPU core
    MsfsSite=127.0.0.1

    #configure for mysql
    DBInstances=teamtalk_master,teamtalk_slave
    #teamtalk_master
    teamtalk_master_host=127.0.0.1
    teamtalk_master_port=3306
    teamtalk_master_dbname=teamtalk
    teamtalk_master_username=root
    teamtalk_master_password=12345
    teamtalk_master_maxconncnt=16

    #teamtalk_slave
    teamtalk_slave_host=127.0.0.1
    teamtalk_slave_port=3306
    teamtalk_slave_dbname=teamtalk
    teamtalk_slave_username=root
    teamtalk_slave_password=12345
    teamtalk_slave_maxconncnt=16


    #configure for unread
    CacheInstances=unread,group_set,token,group_member
    #未读消息计数器的redis
    unread_host=127.0.0.1
    unread_port=6379
    unread_db=1
    unread_maxconncnt=16

    #群组设置redis
    group_set_host=127.0.0.1
    group_set_port=6379
    group_set_db=2
    group_set_maxconncnt=16

    #deviceToken redis
    token_host=127.0.0.1
    token_port=6379
    token_db=4
    token_maxconncnt=16

    #GroupMember
    group_member_host=127.0.0.1
    group_member_port=6379
    group_member_db=5
    group_member_maxconncnt=48

    #AES 密钥
    aesKey=12345678901234567890123456789012

ListenIP:db_proxy_server监听的IP。

ListenPort:db_proxy_server监听的port

ThreadNum:工作线程个数。

MsfsSite:配置msfs服务器的地址，用于发送语音的时候上传保存语音文本。

DBInstances:db实例名称。一般配置一主一从即可，其他根据自己的需求修改。

(xxxx)_host:xxxx实例的ip

(xxxx)_port:xxxx实例的port

(xxxx)_dbname:xxxx实例的scheme名称

(xxxx)_username:xxxx实例的用户名

(xxxx)_password:xxxx实例的密码

(xxxx)_maxconncnt:xxxx实例最大连接数

CacheInstances:cache实例名称。

(xxxx)_host:xxxx实例的ip

(xxxx)_port:xxxx实例的port

(xxxx)_db:xxxx实例的db

(xxxx)_maxconncnt:xxxx

aesKey:消息加密密钥。

目前我们db实例配置的一主一从，cache实例配置了5个实例，分别是:

unread:主要用于未读计数。

group_set:群组设置。设置屏蔽群组。

token:主要用于保存ios系统的token。

group_member:保存群成员信息。

参考配置:

    ListenIP=0.0.0.0
    ListenPort=10600
    ThreadNum=48        # double the number of CPU core
    MsfsSite=http://teamtalk.naturalwill.me:8700/

    #configure for mysql
    DBInstances=teamtalk_master,teamtalk_slave
    #teamtalk_master
    teamtalk_master_host=127.0.0.1
    teamtalk_master_port=3306
    teamtalk_master_dbname=teamtalk
    teamtalk_master_username=teamtalk
    teamtalk_master_password=12345
    teamtalk_master_maxconncnt=16

    #teamtalk_slave
    teamtalk_slave_host=127.0.0.1
    teamtalk_slave_port=3306
    teamtalk_slave_dbname=teamtalk
    teamtalk_slave_username=teamtalk
    teamtalk_slave_password=12345
    teamtalk_slave_maxconncnt=16


    #configure for unread
    CacheInstances=unread,group_set,token,sync,group_member
    #未读消息计数器的redis
    unread_host=127.0.0.1
    unread_port=6379
    unread_db=1
    unread_maxconncnt=16

    #群组设置redis
    group_set_host=127.0.0.1
    group_set_port=6379
    group_set_db=1
    group_set_maxconncnt=16

    #同步控制
    sync_host=127.0.0.1
    sync_port=6379
    sync_db=2
    sync_maxconncnt=1

    #deviceToken redis
    token_host=127.0.0.1
    token_port=6379
    token_db=1
    token_maxconncnt=16

    #GroupMember
    group_member_host=127.0.0.1
    group_member_port=6379
    group_member_db=1
    group_member_maxconncnt=48

    #AES 密钥
    aesKey=12345678901234567890123456789012



#### login_server

    ClientListenIP=0.0.0.0      # can use multiple ip, seperate by ';'
    ClientPort=8008
    HttpListenIP=0.0.0.0
    HttpPort=8080
    MsgServerListenIP=0.0.0.0   # can use multiple ip, seperate by ';'
    MsgServerPort=8100
    msfs=http://127.0.0.1:8700/
    discovery=http://127.0.0.1/api/discovery


ClientListenIP:目前已经作废。

ClientPort:与上一个配套，同样作废。

HttpListenIP:供客户端过来获取msg_server及其他参数的接口地址，走http协议。

HttpPort:与上一个配套使用。

MsgServerListenIP:用于监听msg_server上报信息使用。

MsgServerPort:与上一个配套使用。msg_server启动的时候回来连接该ip:port，以上报自己的信息。

在运行过程中，也会实时将自己的信息汇报给login_server。

msfs:小文件存储的地址，该配置是提供给客户端获取参数时使用。

discovery:发现内容获取地址，该配置是提供给客户端获取参数时使用。


参考配置:

    ClientListenIP=0.0.0.0
    ClientPort=8008
    HttpListenIP=0.0.0.0
    HttpPort=8080
    MsgServerListenIP=0.0.0.0
    MsgServerPort=8100
    msfs=http://teamtalk.naturalwill.me:8700/
    discovery=http://teamtalk.naturalwill.me/api/discovery

#### route_server

    ListenIP=0.0.0.0            # Listening IP
    ListenMsgPort=8200          # Listening Port for MsgServer

route_server配置比较简单，一个监听ip，一个监听port就OK了，供msg_server连接上来用。

参考配置:

    ListenIP=0.0.0.0  
    ListenMsgPort=8200

#### http_msg_server

    ListenIP=0.0.0.0
    ListenPort=8400
    ConcurrentDBConnCnt=4
    DBServerIP1=127.0.0.1
    DBServerPort1=10600
    DBServerIP2=127.0.0.1
    DBServerPort2=10600
    RouteServerIP1=localhost
    RouteServerPort1=8200
    #RouteServerIP2=localhost
    #RouteServerPort2=8201
    
ListenIP: 监听IP，供其他人来调用http_msg_server接口，比如，php在创建群组的时候，就会来调用http_msg_server的接口。

ListenPort: 监听端口，与上一个配套使用。

ConcurrentDBConnCnt: DB数目，目前必须配置为2的整数倍，是历史遗留问题，后期会修复。

DBServerIP(x):db_proxy_server监听的IP，http_msg_server会主动去连接。

DBServerPort(x):db_proxy_server监听的Port

RouteServerIP(x):route_server监听的IP，http_msg_server会主动去连接。

RouteServer(x):route_server监听的Port


参考配置:

    ListenIP=0.0.0.0
    ListenPort=8400
    ConcurrentDBConnCnt=4
    DBServerIP1=127.0.0.1
    DBServerPort1=10600
    DBServerIP2=127.0.0.1
    DBServerPort2=10600
    RouteServerIP1=127.0.0.1
    RouteServerPort1=8200
    #RouteServerIP2=localhost
    #RouteServerPort2=8201

#### msg_server

    ListenIP=0.0.0.0
    ListenPort=8000

    ConcurrentDBConnCnt=2
    DBServerIP1=127.0.0.1
    DBServerPort1=10600
    DBServerIP2=127.0.0.1
    DBServerPort2=10600

    LoginServerIP1=127.0.0.1
    LoginServerPort1=8100
    #LoginServerIP2=localhost
    #LoginServerPort2=8101

    RouteServerIP1=127.0.0.1
    RouteServerPort1=8200
    #RouteServerIP2=localhost
    #RouteServerPort2=8201

    PushServerIP1=127.0.0.1
    PushServerPort1=8500

    FileServerIP1=127.0.0.1
    FileServerPort1=8600
    #FileServerIP2=localhost
    #FileServerPort2=8601

    IpAddr1=127.0.0.1   #电信IP
    IpAddr2=127.0.0.1   #网通IP
    MaxConnCnt=100000

    #AES 密钥
    aesKey=12345678901234567890123456789012
    
    

ListenIP:监听客户端连接上来的IP。

ListenPort:与上一个配套使用，监听客户端连接的port。

ConcurrentDBConnCnt:db_proxy_server个数，同http_msg_server 一样。

DBServerIP(x):db_proxy_server监听的ip，msg_server主动去连接。

DBServerPort(x):db_proxy_server监听的port。

LoginServerIP(x):login_server监听的ip，msg_server会主动去连接，汇报本机信息。

LoginServerPort(x):login_server监听的port。

RouteServerIP(x):route_server监听的IP，msg_server主动去连接。

RouteServerPort(x):route_server监听的port。

PushServerIP(x):push_server监听的IP，msg_server会主动去连接，给ios系统推送消息。

PushServerPort(x):push_server监听的port。

FileServerIP(x):file_server监听的IP，msg_server会主动去连接，用于文件传输，暂时未用到。

FileServerPort(x):file_server监听的port。

IpAddr1:msg_server监听的ip，用于汇报给login_server，便于login_server在客户端请求的时候返回给客户端。注意，这个ip一定要是客户端能连接的ip，之前发现好多人配置成127.0.0.1，这是不行的。

IpAddr2:同上。

aesKey:消息文本加密密钥.这里配置主要在msg_server向push_server推送的时候需要将加密的消息进行解密。


参考配置:

    ListenIP=0.0.0.0
    ListenPort=8000

    ConcurrentDBConnCnt=1
    DBServerIP1=127.0.0.1
    DBServerPort1=10600
    DBServerIP2=127.0.0.1
    DBServerPort2=10600

    LoginServerIP1=192.168.1.70
    LoginServerPort1=8100
    #LoginServerIP2=localhost
    #LoginServerPort2=8101

    RouteServerIP1=192.168.1.70
    RouteServerPort1=8200
    #RouteServerIP2=localhost
    #RouteServerPort2=8201

    PushServerIP1=192.168.1.70
    PushServerPort1=8500

    FileServerIP1=192.168.1.70
    FileServerPort1=8600
    #FileServerIP2=localhost
    #FileServerPort2=8601

    IpAddr1=teamtalk.naturalwill.me #电信IP
    IpAddr2=teamtalk.naturalwill.me #网通IP
    MaxConnCnt=100000

    # AES key
    aesKey=12345678901234567890123456789012

    
#### file_server

    #Address=0.0.0.0         # address for client
    ClientListenIP=0.0.0.0
    ClientListenPort=8600   # Listening Port for client

    MsgServerListenIP=0.0.0.0
    MsgServerListenPort=8601

    TaskTimeout=60         # Task Timeout (seconds)

#### push_server

    ListenIP=0.0.0.0
    ListenPort=8500

    CertPath=apns-dev-cert.pem
    KeyPath=apns-dev-key.pem
    KeyPassword=tt@mogujie

    #SandBox
    #1: sandbox 0: production
    SandBox=1   
    
#### msfs

    BaseDir=/var/www/teamtalk/msfs #文件存放地址
    FileCnt=10
    FilesPerDir=30000
    GetThreadCount=32
    ListenIP=0.0.0.0
    ListenPort=8700
    PostThreadCount=1


    
## 运行

服务端的启动没有严格的先后流程，因为各端在启动后会去主动连接其所依赖的服务端，如果相应的服务端还未启动，会始终尝试连接。不过在此，如果是线上环境,还是建议按照如下的启动顺序去启动(也不是唯一的顺序)：

1、启动 db_proxy。

2、启动 route_server, file_server, msfs

3、启动 login_server

4、启动 msg_server

命令：

```sh
cd db_proxy_server && daeml db_proxy_server && cd ..

cd route_server && daeml route_server && cd ..
cd file_server && daeml file_server && cd ..
cd msfs && daeml msfs && cd ..

cd login_server && daeml login_server && cd ..

cd push_server && daeml push_server && cd ..

cd msg_server && daeml msg_server && cd ..

cd http_msg_server && daeml http_msg_server && cd ..
```


## 参考


蓝狐 的 [ 新版TeamTalk部署教程 ]( http://www.bluefoxah.org/teamtalk/new_tt_deploy.html )
    
luoxn28 的 [ TeamTalk源码分析之服务端描述 ]( http://www.cnblogs.com/luoxn28/p/5348649.html )
