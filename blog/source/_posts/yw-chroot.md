---
title: 运维 Chroot限制普通用户登录特定目录
date: 2017-04-07 22:18:33
tags:
  - chroot
categories:
  - 运维
---
#### 需求
普通用户登陆到服务器上只能执行ssh，ls，cat等有限的基础命令，另外要求把用户锁定在特定目录中，不能看到其他任何目录下文件

#### 实现
1. 创建查看日志的用户
    ```python
    useradd -m ttxsgoto -s /bin/bash
    passwd ttxsgoto
    ```

2. 初始化chroot环境

    ```python
    mkdir /home/chroot
    mkdir /home/chroot/{bin,dev,lib,lib64,etc,home}
    CMD="/bin/bash /bin/ls /bin/cp /bin/mkdir /bin/mv /bin/rm /bin/rmdir /usr/bin/vim /bin/cat /usr/bin/tail"
    CHROOT="/home/chroot"
    lib1=`ldd $CMD | awk '{ print $1 }' | grep "/lib" | sort | uniq`
    lib2=`ldd $CMD | awk '{ print $3 }' | grep "/lib" | sort | uniq`
    for i in $CMD
    do
        cp -a $i $CHROOT/bin/ && echo "$i done"
    done
    for j in $lib1
    do
    cp -f $j $CHROOT/lib64/ && cp -f $j $CHROOT/lib/  && echo "$j done"
    done
    for k in $lib2
    do
    cp -f $k $CHROOT/lib64/ && cp -f $k $CHROOT/lib/ && echo "$k done"
    done
    chown -R root:root /home/chroot
    chmod -R 755 /home/chroot
    ```
3. 创建用户目录
    ```python
    mkdir /home/chroot/home/ttxsgoto
    chown -R ttxsgoto:ttxsgoto /home/chroot/home/ttxsgoto
    ```
4. 添加sshd_config
    ```python
    Match User ttxsgoto
    ChrootDirectory /home/chroot
    ```
5. 日志目录挂载
    ```python
    mount --bind /var/logs /home/chroot/home/ttxsgoto
    #修改/etc/fstab文件，开机自挂载：
    echo "/var/logs     /home/chroot/home/loglooker    none    rw,bind        0    0" >> /etc/fstab
    ```
