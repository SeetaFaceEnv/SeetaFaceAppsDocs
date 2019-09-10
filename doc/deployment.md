# 部署文档

## 注意事项

此部署文档的使用系统为CentOS7



## 代码放置

``` bash
#递归创建项目目录
$ mkrir -p /home/www-root/SeetaAiBuildingCommunity/

#将前端、后端、gateway的代码放到此目录下
#将代码文件名重命名成相应的名字
$ mv <前端代码文件夹名字> web 
$ mv <后端代码文件夹名字> php

#将文件的所属用户改为普通用户
$ chown <OWNER>:<GROUP> -R web

示例：
$ chown seeta:seeta -R web
$ chown seeta:seeta -R php
$ chown seeta:seeta -R gateway

#修改前端配置文件
$ vim /home/www-root/SeetaAiBuildingCommunity/web/config.js
#将这一行中的 '192.168.0.16' 替换成当前服务器的ip
baseURL: 'http://192.168.0.16:9090/'
```



## 更换网易源

```bash
$ wget -O /etc/yum.repos.d/CentOS7-Base-163.repo http://mirrors.163.com/.help/CentOS7-Base-163.repo
$ yum makecache
$ yum list
```



## 安装依赖

```bash
$ yum groupinstall "Development Tools"
$ yum install openssh-server vim openssl-devel libjpeg-devel libpng-devel libtiff-devel jasper-libs jasper-devel atlas-devel lapack-devel blas-devel protobuf-devel openexr-devel ilmbase-devel git yasm bash-completion wget unzip 
```



## PHP7

**卸载旧版**

```bash
$ rpm -qa|grep php
$ yum remove php*
```

**添加centos php7.x的php源**

```bash
$ rpm -Uvh https://mirror.webtatic.com/yum/el7/epel-release.rpm
$ rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
```

**安装**

```bash
$ yum install php70w-common php70w-cli php70w-devel php70w-fpm php70w php70w-pecl-mongodb.x86_64 php70w-pecl-redis.x86_64 php70w-pdo php70w-mbstring php70w-mcrypt php70w-mysqlnd php70w-redis php70w-bcmath php70w-gd
```



## Nginx

为yum添加nginx, 新建文件`/etc/yum.repos.d/nginx.repo`输入：

```ini
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```

**安装**

```bash
$ yum install nginx
```

**配置**

修改nginx参数文件`/etc/nginx/nginx.conf`

```nginx
user hzhs;		#填写当前服务器存在的用户
worker_processes auto;
pid /run/nginx.pid;

events {
        worker_connections 768;
        # multi_accept on;
}

http {
        ##
        # Basic Settings
        ##
        sendfile        on;
        fastcgi_connect_timeout 300s;
        fastcgi_send_timeout 300s;
        fastcgi_read_timeout 300s;
        keepalive_timeout 65;
        client_max_body_size 512M;
        access_log off;
        tcp_nopush on;
        tcp_nodelay on;
        types_hash_max_size 1024;
        # server_tokens off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # Logging Settings
        ##
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##
        gzip on;
        gzip_disable "msie6";

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
}
```

修改nginx参数文件`/etc/nginx/conf.d/SeetaAiBuilding.conf`

```nginx
server {
    listen 9090;
    server_name localhost;
    
    #charset koi8-r;
    error_log   /var/log/seeta_ai_building.error.log;

    location / {
        index       index.php index.html index.htm;
        try_files $uri $uri/ /index.html;
        root        /home/www-root/SeetaAiBuildingCommunity/web;
    }

    #后台路由匹配
    location ~* /(frontend|backend) {
        try_files $uri $uri/ /index.php?_url=$uri&$args;
    }

    #PHP 业务层
    location ~\.php$ {
        root           /home/www-root/SeetaAiBuildingCommunity/php/public;
        include fastcgi_params;
        fastcgi_pass /var/run/php-fpm/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    location ~ /\.ht {
        deny  all;
    }
}

```

## Phalcon3.4

[phalcon3.4下载地址链接](https://pan.baidu.com/s/1l5hxqvTIUwDNyjllcsflvQ)

下载此链接名为`cphalcon.rar`的压缩包，将文件解压放到`/root`目录下

```bash
$ chmod -R 777 ./cphalcon	 #增加可执行权限
$ cd ./cphalcon/build
$ ./install  --php-config /usr/bin/php-config  --phpize /usr/bin/phpize
```

**配置**

编辑`/etc/php.d/phalcon.ini ` ,写入以下内容：

```ini
extension=phalcon.so
```

编辑`/etc/php-zts.d/phalcon.ini`，写入以下内容：

```ini 
extension=phalcon.so
```

修改` /etc/php-fpm.d/www.conf `   //以下下修改需要和nginx的里面一致

```ini
user = hzhs				#填写当前服务器存在的用户
group = hzhs			#填写当前服务器存在的组
listen = /run/php-fpm/php-fpm.sock
listen.owner = hzhs
listen.group = hzhs
listen.mode = 0660
```

修改/etc/php.ini     //修改上传文件大小的方法（根据实际情况修改）

```ini
upload_max_filesize = 512M
post_max_size = 512M
max_execution_time = 300
max_input_time = 300
memory_limit = 512M
date.timezone = "Asia/Shanghai"
```

**重启php**

```bash
$ systemctl restart php-fpm.service
```



## Mongodb

新建文件 `/etc/yum.repos.d/mongodb-org-3.6.repo` ，并写入

```ini
[mongodb-org-3.6]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.6/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.6.asc
```

**安装**

```bash
$ yum install mongodb-org
```

**安装mongoDB的php扩展**

```bash
$ wget https://pecl.php.net/get/mongodb-1.5.2.tgz
$ tar -zxvf mongodb-1.5.2.tgz
$ cd mongodb-1.5.2
$ phpize
$ ./configure --with-php-config=/usr/bin/php-config
$ make && make install
```

新建文件`/etc/php.d/mongo.ini`，写入

```ini
extension=mongodb.so
```

**配置**

修改参数文件`/etc/mongod.conf`，修改：

```yaml
bindIp: 0.0.0.0
```

**重启Mongo**

`systemctl restart mongod`



## Redis

**方法一：yum安装**

```bash
$ yum install redis
```

**安装redis的php扩展**

```bash
$ wget https://github.com/nicolasff/phpredis/archive/master.zip 
$ unzip master.zip
$ cd phpredis-master
$ phpize
$ ./configure
$ make && make install
```

**配置**

新建文件`/etc/php.d/redis.ini`，写入

```ini
extension=redis.so
```

**方法二：下载安装包**

下载最新稳定版的redis：

```bash
$ cd /root
$ wget https://github.com/antirez/redis/archive/5.0.2.tar.gz
```

安装依赖包：

```bash
$ yum install -y epel-release
$ yum install -y gcc
```

进入下载目录并解压：

```bash
$ tar -xzvf 5.0.2.tar.gz
$ cd redis-5.0.2
$ cd deps
$ make jemalloc
$ make hiredis
$ make linenoise
$ make lua
$ cd ..
$ make
$ make install
```

打开配置文件 `/root/redis-5.0.2/redis.conf`：

```ini
bind 127.0.0.1
#修改为
#bind 127.0.0.1
```

进程在后台运行：

```ini
daemonize no
#修改为
daemonize yes
```

日志输出文件等信息：

```ini
logfile ""
#修改为指定的日志文件
logfile "/var/log/redis/redis.log"
```

将第三步配置好的配置文件复制到指定目录

```bash
$ cp /root/redis-5.0.2/redis.conf /etc/redis.conf
```



## Gateway

**安装扩展**

```bash
$ yum install php-posix
```

将gateway代码放到项目目录下

**启动**

后台启动gateway

```bash
$ cd /home/www-root/SeetaAiBuildingCommunity/gateway	#进入gateway目录
$ screen  						#开启后台
$ php start.php start 			#运行gateway
ctrl+A ctrl+D					#隐藏后台
```

在gateway目录下，运行start.php

```bash
$ php start.php start    		# 可以开启gateway目录下所有项目
$ php start.php stop     		# 关闭所有gateway目录下的项目
$ php start.php status			# 查看所有项目的运行情况
```
