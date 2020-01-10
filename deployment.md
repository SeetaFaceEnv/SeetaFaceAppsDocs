# 部署文档

## 注意事项

此部署文档的使用系统为：CentOS7

此部署文档包含两种部署方式：Dcoker部署 、 物理部署

## 方式一、Docker部署

### 1. docker安装

>  若已安装，请忽略此步骤
```bash
#安装
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
$ sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
$ sudo yum makecache fast


$ yum list docker-ce --showduplicates | sort -r #查看可用版本
$ sudo yum install docker-ce-<VERSION STRING> # 安装19.03版本

#启动
$ sudo systemctl start docker
$ sudo systemctl enable docker
```

### 2. docker-compose部署

>  若已安装，请忽略此步骤
```bash
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

如果docker-compose下载失败，则可以直接上传docker-compose可执行文件到/usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

### 3. 部署硬件设备平台

> 若已部署完成，请忽略此步骤
```shell
# 下载硬件设备平台部署文件
$ git clone https://github.com/SeetaFaceEnv/SeetaDevice.git
# 进入项目目录
$ cd SeetaDevice
```

修改`.env`文件内路径和端口，`resource/config/config_release.yaml`文件内地址、端口和key（mongo的addr端口需和`.env`文件内的mongo端口一致，mqtt的ip需为本机IP地址）

运行程序

```shell
$ docker-compose up -d
```

详细步骤请参考：[中科视拓硬件设备平台](https://github.com/SeetaFaceEnv/SeetaDevice)  中的3.系统部署

### 4. 上传必须文件到服务器

上传docker-compose.yaml,.env,resource文件夹到目标服务器同级目录，如下所示

```bash
current_path
    ├── docker-compose.yaml
    ├── .env
    └── resource
        ├── config
        │    └── config.yaml
        ├──fonts
        │    └── courbd.ttf
        └──templates
             └── config.js
```

### 5. 修改.env文件

```dotenv
# dir
dir_prefix=/home/SeetaDeviceCommunity

# mongo
# docker0地址或服务器地址
mongo_ip=172.17.0.1
# mongo运行端口
mongo_port=27007
# mongo用户名
mongo_user=root
# mongo密码
mongo_password=makenosense
# mongo数据目录
mongo_dir=db

# emq
# 部署服务器ip
mqtt_ip=192.168.0.18
# mqtt的tcp端口
mqtt_tcp=1888
# mqtt的dashboard端口
mqtt_dashboard=18088
# mqtt的websocket端口
mqtt_ws=8088
# mqtt的用户名
mqtt_user=admin
# mqtt的密码
mqtt_password=public

# server
# 设备管理平台社区版镜像
server_image=seetaresearch/seeta_device_community:v1.1.beta
# 服务名称
server_name=SeetaDeviceCommunity
# 服务运行模式，支持debug,test,release
server_mode=release
# 服务运行端口
server_port=6969

# seeta_device
# 设备管理平台部署地址
seeta_device_addr=http://192.168.0.7:7878
# 本程序部署地址
seeta_device_callback=http://192.168.0.18:6969

# server dir
# 日志目录
server_dir_log=logs
# 文件目录
server_dir_data=data
# 通行记录照片目录
server_dir_pass_record=passRecords
# 数据采集目录
server_dir_gather=gather
# 服务配置文件目录
server_dir_resource=./resource
```

### 6. 修改前端配置端口

修改config.js文件

```javascript
window.g = {
  //后端地址 http://<服务器ip>:<服务运行端口>/
  baseURL: 'http://192.168.0.18:6969/'
}
```

### 7. 运行程序

```bash
# 运行程序
$ docker-compose up -d

# 重启程序
$ docker-compose restart

# 关闭程序
$ docker-compose down

# 停止程序
$ docker-compose stop

# 启动停止程序
$ docker-compose start

如果镜像拉取失败则可以直接使用镜像文件到服务器，然后docker load < uplaod_image.tar
```

## 方式二、物理部署

### 1.代码放置

``` bash
#本文档的家目录以/home/www-root/为例
#特别提醒：请将后文中/home/www-root/ 替换为 实际目录路径
$ cd /home/www-root
#创建并进入项目目录
$ mkdir SeetaAiBuildingCommunity/
$ cd SeetaAiBuildingCommunity/
#将程序代码放到此目录下
#将代码文件夹名重命名成相应的名字，即app文件夹
$ mv <程序代码文件夹名字> go
# 进入go文件夹
$ cd go
#修改前端配置文件
$ vim resource/templates/config.js
#将这一行中的 '192.168.0.8' 替换成当前服务器的ip
baseURL: 'http://192.168.0.8:6969/'
```

### 2.更换网易源

```bash
$ wget -O /etc/yum.repos.d/CentOS7-Base-163.repo http://mirrors.163.com/.help/CentOS7-Base-163.repo
$ yum makecache
$ yum list
```

### 3.安装go环境

#### 3.1. [国内镜像网站](https://golang.google.cn/dl/)下载golang Linux安装包

#### 3.2. 上传安装包到目标主机

#### 3.3. 安装环境

```bash
$ tar -C /usr/local -xzf <安装包名称>
$ export PATH=$PATH:/usr/local/go/bin
$ export GO111MODULE=on
$ export GOPROXY=https://goproxy.io

#该命令没出现错误，则安装成功
$ go env
```

### 4. 编译源码

```bash
$ go build SeetaDeviceComunity.go
```

### 5. Mongodb

#### 5.1. 新建文件 `/etc/yum.repos.d/mongodb-org-3.6.repo` ，并写入

```ini
[mongodb-org-3.6]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.6/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.6.asc
```

#### 5.2. 安装

```bash
$ yum install mongodb-org
```

#### 5.3. 配置

修改参数文件`/etc/mongod.conf`，修改：

```yaml
bindIp: 0.0.0.0
```

#### 5.4. 重启Mongo

`systemctl restart mongod`


#### 5.5. 创建用户

mongo：

```shell
use admin
db.createUser({ user: "admin", pwd: "admin", roles: [{ role: "userAdminAnyDatabase", db: "admin" }] })
db.auth("admin","admin")
use db_seeta_ai_building
db.createUser({ user: "hzhs", pwd: "hzhs", roles: [{ role: "dbOwner", db: "db_seeta_ai_building" }] })
```

### 6. 修改配置文件(resource/config/config.yaml)

```yaml
server:
  mode: debug #程序运行模式，支持test,release,debug
  port: 6969 #程序运行端口
  third_report: #通行记录转发第三方接口
  sign: 4cc89aa9b9684076804b7974cc16caf1 #第三方注册签名
  log_cycle: 180 #日志保存时长
mongo: #部署的mongodb信息
  ip: 127.0.0.1
  port: 27017
  db: seeta_device_community
  username:
  password:
mqtt: #SeetaDevice部署的emq信息
  ip: 192.168.0.18
  tcp_port: 1883
  ws_port: 8083
  web_scheme: ws #若后台为https，则改为wss
  username: admin
  password: public
  status_topic: status
  record_topic: record
seeta_device: #设备管理平台信息
  addr: http://192.168.0.7:7878 #设备管理平台监听地址
  callback_addr: http://192.168.0.18:6969 #本程序部署的地址
  timeout: 20 #请求超时时间
path:
  log: logs/ #日志文件夹
  data: data/ #数据文件夹
  pass_record: passRecords/ #通行记录文件夹
  gather: gather/ #数据采集文件夹
  fonts: resource/fonts/ #字体文件夹
```

### 7. 启动程序

```bash
$ nohup ./SeetaDeviceCommunity &
```