# 部署文档

## 注意事项

此部署文档的使用系统为CentOS7



## 方式一、Docker部署

**Docker版本**：v18.09.2

**Docker-compose版本**: v1.24.1

### 1.部署文件放置

递归创建项目目录

```shell
$ mkdir -p /home/www-root/
```
将后端 `php` 代码文件中 `docker_deployment` 文件夹放置到 `/home/www-root/` 目录下

```shell
# 重命名项目文件
$ mv docker_deployment SeetaAiBuildingCommunity
```

### 2.配置文件修改

修改前端配置文件config.js

```shell
$ vim /home/www-root/SeetaAiBuildingCommunity/config.js

#将这一行中的 '192.168.0.8' 替换成当前服务器的ip
baseURL: 'http://192.168.0.18:9191/'
```
修改后端配置文件config.php（若部署了设备管理平台，可直接连接设备管理平台docker的mongo、mqtt容器）

```shell
$ vim /home/www-root/SeetaAiBuildingCommunity/config.php

#此为设备管理平台的地址，请根据需要改成实际的地址。
defined('SYSEND_SERVER') || define('SYSEND_SERVER',"http://192.168.0.7:7879/");

# 以下为mongodb的信息请按照实际情况将username、password改成Mongo实际的账号密码
# db_seeta_ai_building为当前使用数据库名
'database' => [
        'adapter' => 'mongodb',
        'url' => 'mongodb://<username>:<password>@127.0.0.1:27017/db_seeta_ai_building',
        'dbname' => 'db_seeta_ai_building',
        'charset' => 'utf8',
    ],

#将redis信息改成实际服务器信息，若设置了redis的密码，则填写相应的redis密码
'redis' => [
        'prefix' => 'tcp://',
        'host' => '127.0.0.1',
        'port' => 6379,
        'password' => '',   //local server
    ],

#本系统中以emq作为消息中间键，请将端口、账号和密码改成实际配置信息
'mqtt' => [
        'host' => '127.0.0.1',
        'port' => 1883,
        'webPort' => 8083,
        'passwd' => '',
        'user' => '',
    ],
```

### 3.启动项目

```shell
# 进入docker部署文件目录
$ cd /home/www-root/SeetaAiBuildingCommunity
$ docker-compose up -d		 #-d参数：后台运行
```

### 4.创建管理员账号

更改资源文件夹的所属用户组，并初始化管理员账号

```shell
# 进入php容器
$ docker exec -it php_release /bin/bash
# 更改资源文件夹的所属用户组
$ chown www-data:www-data /app/php/files
# 初始化管理员账号
$ php /app/php/app/bootstrap_cli.php main CreateAdmin
```

创建的初始账户，用户名：admin，密码：123456

### 5.其他

其他 docker-compose 命令

```shell
# 查看指定容器状态
$ docker-compose ps
# 停止并删除指定容器
$ docker-compose down
# 启动指定服务已存在的容器
$ docker-compose start
# 停止指定服务已存在的容器
$ docker-compose stop
```



## 方式二、物理部署

### 1.代码放置

``` bash
#递归创建项目目录
$ mkdir -p /home/www-root/SeetaAiBuildingCommunity/

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



### 2.更换网易源

```bash
$ wget -O /etc/yum.repos.d/CentOS7-Base-163.repo http://mirrors.163.com/.help/CentOS7-Base-163.repo
$ yum makecache
$ yum list
```



### 3.安装依赖

```bash
$ yum groupinstall "Development Tools"
$ yum install openssh-server vim openssl-devel libjpeg-devel libpng-devel libtiff-devel jasper-libs jasper-devel atlas-devel lapack-devel blas-devel protobuf-devel openexr-devel ilmbase-devel git yasm bash-completion wget unzip 
```



### 4.PHP7

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

**安装PHP**

```bash
$ yum install php70w-common php70w-cli php70w-devel php70w-fpm php70w php70w-pecl-mongodb.x86_64 php70w-pecl-redis.x86_64 php70w-pdo php70w-mbstring php70w-mcrypt php70w-mysqlnd php70w-redis php70w-bcmath php70w-gd 
```

**安装mqqt所需的mosquitto扩展**

```bash
$ pecl install Mosquitto-0.4.0  #过程需要按一次回车键确认
```

编辑`/etc/php.d/mosquitto.ini ` ,写入以下内容：

```ini
extension=mosquitto.so
```

编辑`/etc/php-zts.d/mosquitto.ini`，写入以下内容：

```ini 
extension=mosquitto.so
```



### 5.Nginx

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

    # 后台路由匹配
    location ~* /(frontend|backend) {
        try_files $uri $uri/ /index.php?_url=$uri&$args;
    }

    # PHP 业务层
    location ~\.php$ {
        root           /home/www-root/SeetaAiBuildingCommunity/php/public;
        include fastcgi_params;
        fastcgi_pass unix:/run/php-fpm/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
    
    # 文件路径转发
    location ~ /files/.*\.(html|htm|gif|jpg|jpeg|bmp|png|ico|txt|js|css|apk)$  {
        root /home/www-root/SeetaAiBuildingCommunity/php;
        autoindex on;    # 开启预览功能
    }


    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    location ~ /\.ht {
        deny  all;
    }
}

```



### 6.Phalcon3.4

phalcon3.4包地址：https://pan.baidu.com/s/1l5hxqvTIUwDNyjllcsflvQ

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



### 7.Mongodb

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

mongo：

```shell
use admin

db.createUser({ user: "admin", pwd: "admin", roles: [{ role: "userAdminAnyDatabase", db: "admin" }] })

db.auth("admin","admin")

use db_seeta_ai_building

db.createUser({ user: "hzhs", pwd: "hzhs", roles: [{ role: "dbOwner", db: "db_seeta_ai_building" }] })
```





### 8.Redis

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

### 9.创建管理员账号

`php目录/app/modules/cli/tasks/MainTask.php` 中的 `createAdminAction` 将会创建初始化的 admin账号

可直接在终端输入命令 ，执行此脚本

```bash
$ php /home/www-root/SeetaAiBuildingCommunity/php/run main createAdmin
```

创建的初始账户，用户名：admin，密码：123456

