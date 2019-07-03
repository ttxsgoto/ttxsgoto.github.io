---
title: Sersync+Rsync文件同步
date: 2019-01-10 21:58:34
tags:
  - sersync
categories:
  - 运维
---

#### 背景
需要将某台服务器目录实时同步到另外一台服务器上的某个目录下，其实本来可以通过远程目录挂载的方式完成， 但是现在的需求是某两台服务器的目录实时同步到另外一台的同一目录中，这时就不能使用远程目录挂载方式完成了， 这里使用sersync+rsync来完成需求

#### Sersync特点
- c++编写，对linux系统文件系统产生的临时文件和重复的文件操作进行过滤，结合rsync同步的时候，节省了运行时耗和网络资源
- 使用多线程进行同步，在同步较大文件时，能够保证多个服务器实时保持同步状态
- 自带crontab功能，只需在xml配置文件中开启，隔一段时间整体同步一次
- 自定义同步规则

#### Rsync
配置文件说明
```python
uid = root
gid = root
port = 873
max connections = 1000
timeout = 600
pid file = /var/run/rsyncd.pid
log file = /var/log/rsync.log
lockfile = /var/run/rsyncd.lock
motd file = /etc/rsyncd/rsyncd.moth
log format = %t %a %m %f %b
 
[yw_test]
path = /data/yw_test
ignore errors = yes
list = no
ignore errors
read only = no
auth users = rsync
secrets file = /etc/rsync/rsyncd.secrets
```
启动
```
chmod 600 /etc/rsync/rsyncd.secrets
rsync --daemon --config=/etc/rsync/rsyncd.conf
```
#### Sersync
配置文件confxml.xml说明
```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<head version="2.5">
    <host hostip="localhost" port="8008"></host>
    <debug start="false"/>
    <fileSystem xfs="false"/>
    <!--监控事件的过程中过滤特定文件，和特定文件夹的文件 --> 
    <filter start="true">
	<exclude expression="log.txt"></exclude>
	<exclude expression="(.*)\.svn"></exclude>
	<exclude expression="(.*)\.gz"></exclude>
    </filter>
    <!--设置要监控的事件 --> 
    <inotify>
	<delete start="false"/>
	<createFolder start="true"/>
	<createFile start="true"/>
	<closeWrite start="true"/>
	<moveFrom start="true"/>
	<moveTo start="true"/>
	<attrib start="false"/>
	<modify start="false"/>
    </inotify>
  
    <sersync>
     <!--设定监控目录--> 
	<localpath watch="/data/test">
		<!--指定远端rsync的地址和模块名-->
	    <remote ip="192.168.0.1" name="yw_test"/>   
	</localpath>
	<rsync>
	    <!--commonParams params="-rczgop"/-->
	    <commonParams params="-artuz"/>
	    <!--是否启用验证，并指定密码存放文件 --> 
	    <auth start="true" users="rsync" passwordfile="/etc/rsync_client.pass "/>
	    <userDefinedPort start="false" port="873"/><!-- port=874 -->
	    <timeout start="false" time="100"/><!-- timeout=100 -->
	    <ssh start="false"/>
	</rsync>
	<failLog path="/tmp/rsync_fail_log.log" timeToExecute="10"/>
	<!--default every 60mins execute once-->
	<!--是否启用执行完整rsync，并指定执行周期 -->     
	<crontab start="true" schedule="5"><!--5mins-->
	    <crontabfilter start="true">
		<exclude expression="log.txt"></exclude>
		<exclude expression="(.*)\.svn"></exclude>
		<exclude expression="(.*)\.gz"></exclude>
		<exclude expression="info/*"></exclude>
	    </crontabfilter>
	</crontab>
	<plugin start="false" name="command"/>
    </sersync>
  
    <plugin name="command">
	<param prefix="/bin/sh" suffix="" ignoreError="true"/>	<!--prefix /opt/tongbu/mmm.sh suffix-->
	<filter start="false">
	    <include expression="(.*)\.php"/>
	    <include expression="(.*)\.sh"/>
	</filter>
    </plugin>
 
    <plugin name="socket">
	<localpath watch="/opt/tongbu">
	    <deshost ip="192.168.138.20" port="8009"/>
	</localpath>
    </plugin>
    <plugin name="refreshCDN">
	<localpath watch="/data0/htdocs/cms.xoyo.com/site/">
	    <cdninfo domainname="ccms.chinacache.com" port="80" username="xxxx" passwd="xxxx"/>
	    <sendurl base="http://pic.xoyo.com/cms"/>
	    <regexurl regex="false" match="cms.xoyo.com/site([/a-zA-Z0-9]*).xoyo.com/images"/>
	</localpath>
    </plugin>
</head>
```

启动
```
chmod 600 /etc/rsync_client.pass 
/etc/rsync/sersync2 -r -d -o /etc/rsync/confxml.xml
 
-d:启用守护进程模式
-r:在监控前，将监控目录与远程主机用rsync命令推送一遍
-n: 指定开启守护线程的数量，默认为10个
-o:指定配置文件，默认使用confxml.xml文件
 
```
