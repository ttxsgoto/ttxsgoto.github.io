---
title: 运维 Centos，Ubuntu下安装zabbix遇到的问题
date: 2017-03-06 22:15:20
tags:
  - Zabbix
categories:
  - 运维
---

#### Ubuntu
在Ubuntu系统中，安装zabbix进行configure时会遇到以下4个主要问题
```
./configure --enable-server --enable-agent --with-mysql --with-net-snmp --with-jabber --with-libcurl
 
1 configure: error: MySQL library not found
the problem is not installed mysql-devel
#apt-get install libghc6-hsql-mysql-dev (ubuntu )
 
2 configure: error: Jabber library not found
the problem is not installed jabber lib
#apt-get install libphp-jabber
#apt-get install libnet-jabber-loudmouth-perl
#apt-get install jabber-dev
#apt-get install libiksemel-dev (* this packet important)
 
3 configure: error: Not found curl Library
the problem is not installed libcurl4-openssl-dev
#apt-get install libcurl4-openssl-dev
 
4 configure: error : Not found NET-SNMP library
#apt-get install libsnmp-dev
#apt-get install snmp  
```
#### CentOS
在CentOS系统中，安装zabbix进行configure时会遇到以下4个主要问题
```
./configure --enable-server --enable-agent --with-mysql --with-net-snmp --with-jabber --with-libcurl
 
1 configure: error: MySQL library not found
the problem is not installed mysql-devel
#yum install mysql-devel
#yum install mysql-server
 
2 configure: error: Jabber library not found
the problem is not installed jabber lib
#wget http://iksemel.googlecode.com/files/iksemel-1.4.tar.gz
下载完成后解压、配置、安装：
tar zxvf iksemel-1.4.tar.gz
cd iksemel-1.4
configure
make
make install
 
之后对zabbix进行configure还是会遇到这个问题，那么将jabber目录指定即可：#./configure --enable-server --enable-agent --with-mysql --with-net-snmp --with-jabber=/usr/local/ --with-libcurl
 
3 configure: error: Not found curl Library
#yum install curl-devel (此项未经测试)
 
4 configure: error : Not found NET-SNMP library
yum install net-snmp-devel
```














