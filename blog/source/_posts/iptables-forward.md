---
title: 运维 Iptables做端口映射
date: 2017-07-21 20:49:34
tags:
  - Iptables
categories:
  - 运维
---
### 需求
外网需要访问内网的redis(无公网ip)服务器A，而它只能通过内网访问，其中服务器B有公网和内网地址，B通过内网可以访问A的服务；这时可以通过端口转发，通过访问B的端口来实际访问A的redis服务

### 解决方案
```bash
# B服务器 外网ip: 120.27.114.114   内网ip: 10.10.10.10 端口:6379  转发
# A服务器 内网ip: 10.10.10.12:6379
echo '1' > /proc/sys/net/ipv4/ip_forward
sysctl -p
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -d 10.10.10.12 -p tcp --dport 6379 -j ACCEPT
iptables -t nat -A PREROUTING -d 120.27.114.114 -p tcp -m tcp --dport 6379 -j DNAT --to-destination 10.10.10.12:6379
iptables -t nat -A POSTROUTING -d 10.10.10.12 -p tcp -m tcp --dport 6379 -j SNAT --to-source 10.10.10.10

iptables-save > /etc/iptables/rules.v4       # 保存
iptables-restore < /etc/iptables/rules.v4	# 导入
```

