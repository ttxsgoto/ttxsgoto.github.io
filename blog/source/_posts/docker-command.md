---
title: Docker 常用命令
date: 2017-02-09 21:08:57
tags:
  - Docker
categories:
  - Docker
---
**主要记录Docker工作中，常使用命令.**

### 常用命令
```
docker build -t name ./path(dockerfile)  #通过dockerfile来创建镜像
    --rm=true  表示构建成功后，移除所有中间容器
    --no-cache=false 表示在构建过程中不使用缓存

docker run  -it -p 2222:22 --name 容器名称 镜像名称  #启动docker镜像
    -it 交互式模式启动，前台可以看到
    -d 后台模式启动
    -p 2222:22指定端口映射
    --name 容器名称
    -v  host_dir:container_dir   宿主机和容器数据映射，数据同步
    -e 传递环境变量  -e WORDPRESS_DB_HOST=x.x.x.x
    --entrypoint=/bin/bash   将cmd命令的环境覆盖掉
如：docker run -itd -p 80:80 --name nginx_php -v /root/html:/data/www/html nginx1.8
docker ps  # 将处于run状态下的容器显示出来
docker ps -a   #将所有docker状态的容器显示出来
docker info     #查看docker信息
docker images # 查看镜像
docker pull   xxx  #下载镜像
docker push  xxx  #将镜像推送到registry
docker search xxxx #查询镜像
docker diff     #列出容器内发生变化的文件和目录（A-Add，D-Delete，C-Change）
docker commit  xxxx    abc/yyy  #把有修改的container提交到新的images中
docker tag  原镜像名称     新名称    #给镜像重命名
docker top container    #查看正在运行的容器中的进程的运行情况
docker port container   #用于查看容器与主机之间的端口映射关系信息
docker exec -it  container_id(base(名称)) /bin/bash    #进入docker容器里面
docker stop b1430f1a3daa    #停止容器运行
Ctrl +p +q                         #进入容器后，从终端退出容器
docker rm  -f 容器名称   #删除容器，-f强制删除
docker rmi  image  #移除一个或多个镜像
docker inspect  #查看镜像或容器的底层详细信息
docker inspect -f {{.NetworkSettings.IPAddress}}  xxxxx   -f #查看特定信息
docker ps -qa  # 列出所有的容器(含沉睡镜像)的容器ID号
docker rm `docker ps -qa` 将沉睡的容器删除
```
### 存储相关命令
```
docker save -o ubuntu_14.04.tar ubuntu:14.04   #存出镜像
docker load < ubuntu_14.04.tar   #载入镜像，导入相关的元数据（包括标签等）
docker export ID(7691a814370e) >ubuntu.tar   #导出容器，导出容器快照到本地
docker import             #导入容器快照，可以导入远程文件、本地文件和目录，使用http的url从远程位置导入，本地或目录的导入需要使用-参数
如：docker import http://xxx.yyy.com/ext.tar.gz  xxx/yyy  || docker import - ubuntu:14.04
```
### 日志相关命令
```
docker events     #打印容器实时的系统事件
docker history  images    #打印指定image的每层image命令行的历史记录
docker logs container  #批量打印出容器中进程的运行日志
```

