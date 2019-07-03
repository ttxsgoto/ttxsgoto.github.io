---
title: Systemd服务
date: 2019-06-29 11:07:07
tags:
  - systemd
categories:
  - 运维
---
### 说明
Systemd 是 Linux 系统工具，用来启动守护进程，已成为大多数发行版的标准配置；主要用于服务管理和自定义服务的管理，方便运维工作管理。

#### systemctl
systemctl是 Systemd 的主命令，用于管理系统
```
# 重启系统
$ sudo systemctl reboot
 
# 关闭系统，切断电源
$ sudo systemctl poweroff
 
# CPU停止工作
$ sudo systemctl halt
 
# 暂停系统
$ sudo systemctl suspend
 
# 让系统进入冬眠状态
$ sudo systemctl hibernate
 
# 让系统进入交互式休眠状态
$ sudo systemctl hybrid-sleep
 
# 启动进入救援状态（单用户状态）
$ sudo systemctl rescue
```
#### systemd-analyze
systemd-analyze命令用于查看启动耗时
```
# 查看启动耗时
$ systemd-analyze
 
# 查看每个服务的启动耗时
$ systemd-analyze blame
 
# 显示瀑布状的启动过程流
$ systemd-analyze critical-chain
 
# 显示指定服务的启动流
$ systemd-analyze critical-chain atd.service
```
#### hostnamectl
hostnamectl命令用于查看当前主机的信息
```
# 显示当前主机的信息
$ hostnamectl
 
# 设置主机名
$ sudo hostnamectl set-hostname test01
```
#### timedatectl
timedatectl命令用于查看当前时区设置
```
# 查看当前时区设置
$ timedatectl
 
# 显示所有可用的时区
$ timedatectl list-timezones                                                        
# 设置当前时区
$ sudo timedatectl set-timezone America/New_York
$ sudo timedatectl set-time YYYY-MM-DD
$ sudo timedatectl set-time HH:MM:SS
```
#### loginctl
loginctl命令用于查看当前登录的用户
```
# 列出当前session
$ loginctl list-sessions
 
# 列出当前登录用户
$ loginctl list-users
 
# 列出显示指定用户的信息
$ loginctl show-user test
```

### Unit
Systemd 可以管理所有系统资源，不同的资源统称为 Unit（单位）
```
Service unit：系统服务
Target unit：多个 Unit 构成的一个组
Device Unit：硬件设备
Mount Unit：文件系统的挂载点
Automount Unit：自动挂载点
Path Unit：文件或路径
Scope Unit：不是由 Systemd 启动的外部进程
Slice Unit：进程组
Snapshot Unit：Systemd 快照，可以切回某个快照
Socket Unit：进程间通信的 socket
Swap Unit：swap 文件
Timer Unit：定时器
```
systemctl list-units命令可以查看当前系统的所有 Unit 
```
# 列出正在运行的 Unit
$ systemctl list-units
 
# 列出所有Unit，包括没有找到配置文件的或者启动失败的
$ systemctl list-units --all
 
# 列出所有没有运行的 Unit
$ systemctl list-units --all --state=inactive
 
# 列出所有加载失败的 Unit
$ systemctl list-units --failed
 
# 列出所有正在运行的、类型为 service 的 Unit
$ systemctl list-units --type=service
```

#### Unit 的状态
systemctl status命令用于查看系统状态和单个 Unit 的状态
```
# 显示系统状态
$ systemctl status
 
# 显示单个 Unit 的状态
$ sysystemctl status test.service
  
# 显示远程主机的某个 Unit 的状态
$ systemctl -H root@localhost status httpd.service
除了status命令，systemctl还提供了三个查询状态的简单方法，主要供脚本内部的判断语句使用。
  
# 显示某个 Unit 是否正在运行
$ systemctl is-active test.service
 
# 显示某个 Unit 是否处于启动失败状态
$ systemctl is-failed test.service
 
# 显示某个 Unit 服务是否建立了启动链接
$ systemctl is-enabled test.service
```
#### Unit 管理
最常用的是下面这些命令，用于启动和停止 Unit（主要是 service）
```
# 立即启动一个服务
$ sudo systemctl start apache.service
 
# 立即停止一个服务
$ sudo systemctl stop apache.service
 
# 重启一个服务
$ sudo systemctl restart apache.service
 
# 杀死一个服务的所有子进程
$ sudo systemctl kill apache.service
 
# 重新加载一个服务的配置文件
$ sudo systemctl reload apache.service
 
# 重载所有修改过的配置文件
$ sudo systemctl daemon-reload
 
# 显示某个 Unit 的所有底层参数
$ systemctl show httpd.service
 
# 显示某个 Unit 的指定属性的值
$ systemctl show -p CPUShares httpd.service
 
# 设置某个 Unit 的指定属性
$ sudo systemctl set-property httpd.service CPUShares=500
```
#### 依赖关系
Unit 之间存在依赖关系：A 依赖于 B，就意味着 Systemd 在启动 A 的时候，同时会去启动 B
systemctl list-dependencies命令列出一个 Unit 的所有依赖
```
$ systemctl list-dependencies nginx.service
上面命令的输出结果之中，有些依赖是 Target 类型（详见下文），默认不会展开显示。如果要展开 Target，就需要使用--all参数。
 
$ systemctl list-dependencies --all nginx.service
```

### Unit 配置文件
每一个 Unit 都有一个配置文件，告诉 Systemd 怎么启动这个Unit,Systemd 默认从目录/etc/systemd/system/读取配置文件。但是，里面存放的大部分文件都是符号链接，指向目录/usr/lib/systemd/system/，真正的配置文件存放在那个目录。
systemctl enable命令用于在上面两个目录之间，建立符号链接关系。
```
sudo systemctl enable test.service 
Created symlink from /etc/systemd/system/multi-user.target.wants/test.service to /usr/lib/systemd/system/test.service.
```
如果配置文件里面设置了开机启动，systemctl enable命令相当于激活开机启动。
与之对应的，systemctl disable命令用于在两个目录之间，撤销符号链接关系，相当于撤销开机启动。
```
sudo systemctl disable test.service 
```
配置文件的后缀名，就是该 Unit 的种类，比如sshd.socket。如果省略，Systemd 默认后缀名为.service，所以sshd会被理解成sshd.service。

#### 配置文件的状态
systemctl list-unit-files命令用于列出所有配置文件
```
# 列出所有配置文件
$ systemctl list-unit-files
UNIT FILE              STATE
chronyd.service        enabled
clamd@.service         static
clamd@scan.service     disabled
 
enabled：已建立启动链接
disabled：没建立启动链接
static：该配置文件没有[Install]部分（无法执行），只能作为其他配置文件的依赖
masked：该配置文件被禁止建立启动链接
 
# 列出指定类型的配置文件
$ systemctl list-unit-files --type=service
```
从配置文件的状态无法看出，该 Unit 是否正在运行。这必须执行前面提到的systemctl status命令
```
 systemctl status test.service
```
一旦修改配置文件，就要让 SystemD 重新加载配置文件，然后重新启动，否则修改不会生效
```
$ sudo systemctl daemon-reload
$ sudo systemctl restart httpd.service
```
#### 配置文件格式
systemctl cat命令可以查看配置文件的内容
```
$ systemctl cat test.service
 
[Unit]
Description=Test Demo
After=syslog.target network.target
 
[Service]
ExecStart=/usr/bin/python2.7 /home/user/python/test.py
 
[Install]
WantedBy=multi-user.target
```
#### 配置文件说明
[Unit]区块通常是配置文件的第一个区块，用来定义 Unit 的元数据，以及配置与其他 Unit 的关系。它的主要字段如下
```
Description：简短描述
Documentation：文档地址
Requires：当前 Unit 依赖的其他 Unit，如果它们没有运行，当前 Unit 会启动失败
Wants：与当前 Unit 配合的其他 Unit，如果它们没有运行，当前 Unit 不会启动失败
BindsTo：与Requires类似，它指定的 Unit 如果退出，会导致当前 Unit 停止运行
Before：如果该字段指定的 Unit 也要启动，那么必须在当前 Unit 之后启动
After：如果该字段指定的 Unit 也要启动，那么必须在当前 Unit 之前启动
Conflicts：这里指定的 Unit 不能与当前 Unit 同时运行
Condition...：当前 Unit 运行必须满足的条件，否则不会运行
Assert...：当前 Unit 运行必须满足的条件，否则会报启动失败
```
[Install]通常是配置文件的最后一个区块，用来定义如何启动，以及是否开机启动。它的主要字段如下
```
WantedBy：它的值是一个或多个 Target，当前 Unit 激活时（enable）符号链接会放入/etc/systemd/system目录下面以 Target 名 + .wants后缀构成的子目录中
RequiredBy：它的值是一个或多个 Target，当前 Unit 激活时，符号链接会放入/etc/systemd/system目录下面以 Target 名 + .required后缀构成的子目录中
Alias：当前 Unit 可用于启动的别名
Also：当前 Unit 激活（enable）时，会被同时激活的其他 Unit
```
[Service]区块用来 Service 的配置，只有 Service 类型的 Unit 才有这个区块。它的主要字段如下
```
Type：定义启动时的进程行为。它有以下几种值。
Type=simple：默认值，执行ExecStart指定的命令，启动主进程
Type=forking：以 fork 方式从父进程创建子进程，创建后父进程会立即退出
Type=oneshot：一次性进程，Systemd 会等当前服务退出，再继续往下执行
Type=dbus：当前服务通过D-Bus启动
Type=notify：当前服务启动完毕，会通知Systemd，再继续往下执行
Type=idle：若有其他任务执行完毕，当前服务才会运行
ExecStart：启动当前服务的命令
ExecStartPre：启动当前服务之前执行的命令
ExecStartPost：启动当前服务之后执行的命令
ExecReload：重启当前服务时执行的命令
ExecStop：停止当前服务时执行的命令
ExecStopPost：停止当其服务之后执行的命令
RestartSec：自动重启当前服务间隔的秒数
Restart：定义何种情况 Systemd 会自动重启当前服务，可能的值包括always（总是重启）、on-success、on-failure、on-abnormal、on-abort、on-watchdog
TimeoutSec：定义 Systemd 停止当前服务之前等待的秒数
Environment：指定环境变量
```

### 日志管理
Systemd 统一管理所有 Unit 的启动日志。带来的好处就是，可以只用journalctl一个命令，查看所有日志（内核日志和应用日志）。日志的配置文件是/etc/systemd/journald.conf。
```
# 查看所有日志（默认情况下 ，只保存本次启动的日志）
$ sudo journalctl
 
# 查看内核日志（不显示应用日志）
$ sudo journalctl -k
 
# 查看系统本次启动的日志
$ sudo journalctl -b
$ sudo journalctl -b -0
 
# 查看上一次启动的日志（需更改设置）
$ sudo journalctl -b -1
 
# 查看指定时间的日志
$ sudo journalctl --since="2012-10-30 18:17:16"
$ sudo journalctl --since "20 min ago"
$ sudo journalctl --since yesterday
$ sudo journalctl --since "2015-01-10" --until "2015-01-11 03:00"
$ sudo journalctl --since 09:00 --until "1 hour ago"
 
# 显示尾部的最新10行日志
$ sudo journalctl -n
 
# 显示尾部指定行数的日志
$ sudo journalctl -n 20
 
# 实时滚动显示最新日志
$ sudo journalctl -f
 
# 查看指定服务的日志
$ sudo journalctl /usr/lib/systemd/systemd
 
# 查看某个路径的脚本的日志
$ sudo journalctl /usr/bin/bash
 
# 查看指定用户的日志
$ sudo journalctl _UID=12 --since today
 
# 查看某个 Unit 的日志
$ sudo journalctl -u nginx.service
$ sudo journalctl -u nginx.service --since today
 
# 实时滚动显示某个 Unit 的最新日志
$ sudo journalctl -u nginx.service -f
 
# 合并显示多个 Unit 的日志
$ journalctl -u nginx.service -u php-fpm.service --since today
 
# 查看指定优先级（及其以上级别）的日志，共有8级
# 0: emerg
# 1: alert
# 2: crit
# 3: err
# 4: warning
# 5: notice
# 6: info
# 7: debug
$ sudo journalctl -p err -b
 
# 日志默认分页输出，--no-pager 改为正常的标准输出
$ sudo journalctl --no-pager
 
# 以 JSON 格式（单行）输出
$ sudo journalctl -b -u nginx.service -o json
 
# 以 JSON 格式（多行）输出，可读性更好
$ sudo journalctl -b -u nginx.serviceqq
 -o json-pretty
 
# 显示日志占据的硬盘空间
$ sudo journalctl --disk-usage
 
# 指定日志文件占据的最大空间
$ sudo journalctl --vacuum-size=1G
 
# 指定日志文件保存多久
$ sudo journalctl --vacuum-time=1years
```
### 开机启动
对于那些支持 Systemd 的软件，安装的时候，会自动在/usr/lib/systemd/system目录添加一个配置文件。

如果你想让该软件开机启动，就执行下面的命令（以httpd.service为例）。
```
$ sudo systemctl enable httpd
```
上面的命令相当于在/etc/systemd/system目录添加一个符号链接，指向/usr/lib/systemd/system里面的httpd.service文件。

这是因为开机时，Systemd只执行/etc/systemd/system目录里面的配置文件。这也意味着，如果把修改后的配置文件放在该目录，就可以达到覆盖原始配置的效果。

### 修改配置文件后重启
修改配置文件以后，需要重新加载配置文件，然后重新启动相关服务
```
# 重新加载配置文件
$ sudo systemctl daemon-reload
 
# 重启相关服务
$ sudo systemctl restart test.service
```
