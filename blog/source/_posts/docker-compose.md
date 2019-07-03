---
title: Docker Compose
date: 2017-02-11 09:38:57
tags:
  - docker-compose
categories:
  - Docker
---
#### 功能
Docker Compose 是Docker容器进行编排的工具，定义和运行多容器的应用，可以一条命令启动多个容器.
#### 安装
```bash
curl -L https://github.com/docker/compose/releases/download/1.6.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose -version
```
#### 常用命令
需要在docker-compose.yml存在的当前目录中
```bash
docker-compose up -d    #启动
docker-compose ps   #查看容器列表
--verbose：输出详细信息
-f 制定一个非docker-compose.yml命名的yaml文件
-p 设置一个项目名称（默认是directory名）
 
build：构建服务
kill -s SIGINT：给服务发送特定的信号。
logs：输出日志
port：输出绑定的端口
ps：输出运行的容器
pull：pull服务的image
rm：删除停止的容器
run: 运行某个服务，例如docker-compose run web python manage.py shell
start：运行某个服务中存在的容器
stop:停止某个服务中存在的容器
restart:重启相关容器
up：create + run + attach容器到服务
scale：设置服务运行的容器数量。例如：docker-compose scale web=2 worker=3
```
#### yml语法
[compose-file](https://docs.docker.com/compose/compose-file/)
```bash
image ：镜像id
build:直接从pwd的Dockerfile来build，而非通过image选项来pull
container_name：容器名称
restart: always 重启
links：连接到那些容器,该yaml文件中的容器，会将相关信息存放在/etc/hosts
external_links：连接到该compose.yaml文件之外的容器中，同links
depends_on：依赖关系说明
command：执行command命令
ports：导出端口，如：ports:-"3000"-"8000:8000"-"127.0.0.1:8001:8001"
expose：导出端口，但不映射到宿主机的端口上。它仅对links的容器开放
volumes：加载路径作为卷，可以指定只读模式
volumes_from：加载其他容器或者服务的所有卷
env_file：从一个文件中导入环境变量，文件的格式为RACK_ENV=development
environment:设置环境变量的值
extends:扩展另一个服务，可以覆盖其中的一些选项
net：容器的网络模式，可以为”bridge”, “none”, “container:[name or id]”, “host”中的一个
dns：可以设置一个或多个自定义的DNS地址
dns_search：dns搜索的域名
logging：日志配置
```
#### 实例
```bash
# https://github.com/vmware/harbor
version: '2'    #指定compose版本
services:
  log:    #服务名称
    image: vmware/harbor-log    #指定镜像名称
    container_name: harbor-log  #启动后的容器名称
    restart: always    #down掉自动重启
    volumes:    #宿主机和容器关联的目录
      - /var/log/harbor/:/var/log/docker/
    ports:    #映射出来的端口
      - 1514:514
  registry:
    image: library/registry:2.5.0
    container_name: registry
    restart: always
    volumes:
      - /data/registry:/storage
      - ./common/config/registry/:/etc/registry/
    environment:    #设置环境变量
      - GODEBUG=netdns=cgo
    command:    #容器内执行命令
      ["serve", "/etc/registry/config.yml"]
    depends_on:    #依赖关系
      - log
    logging:    #日志设置
      driver: "syslog"    #指定日志设备的容器
      options:  
        syslog-address: "tcp://127.0.0.1:1514" #日志连接地址
        tag: "registry"    #日志标签
```







