---
title: 运维 Linux中密码策略
date: 2017-04-05 22:02:10
tags:
  - password
categories:
  - 运维
---
#### 需求
1. 密码必须符合复杂度要求，字母、数字、特殊字符组成。长度大于8位。
2. 密码定期更改，最长不超过90天。
3. 用户不能重复使用5次内已使用的口令。
4. 尝试登陆失败错误次数，必须设置不能超过5次，超过5次后，暂时锁定20分钟或以上。

#### 实现
安装 PAM 的 cracklib 模块，cracklib 能提供额外的密码检查能力

1. 密码必须符合复杂度要求，字母、数字、特殊字符组成。长度大于8位。
修改文件：/ect/pam.d/system-auth，找到同时有 “password” 和 “pam_cracklib.so” 字段
```
password    requisite     pam_cracklib.so retry=3 difok=3 minlen=8 lcredit=-1 dcredit=-1 ocredit=-1
```

2. 密码定期更改，最长不超过90天

	配置文件中 /etc/login.defs 修改配置文件
```
PASS_MAX_DAYS   90
PASS_MIN_DAYS   0
PASS_MIN_LEN    5
PASS_WARN_AGE   7
```
	通过chage -l xxx(用户名)  查看定期修改的时间

3. 用户不能重复使用5次内已使用的口令
修改文件：/ect/pam.d/system-auth，找到同时有 “password” 和 “pam_unix.so” 字段
```
password    sufficient    pam_unix.so sha512 shadow nullok try_first_pass use_authtok remember=5
```
	通过/etc/security/opasswd中查看禁止使用近期用过的5个密码

4. 尝试登陆失败错误次数，必须设置不能超过5次，超过5次后，暂时锁定20分钟或以上
查看系统中是否含有pam_tally2.so模块，如果没有，则需要使用pam_tally.so模块
find /lib* -iname "pam_tally2.so"
find /lib* -iname "pam_tally.so"
在配置文件中/etc/pam.d/sshd中的第二行，如下添加内容
```
auth       required     pam_tally2.so deny=5 unlock_time=1200
```
    查看用户错误登陆次数：
    pam_tally2 --user xxx (用户名)

    ```
    pam_cracklib.so比较重要和难于理解的是它的一些参数和计数方法，其常用参数包括:
    debug：将调试信息写入日志；
    type=xxx：当添加/修改密码时，系统给出的缺省提示符是“New UNIX password:”以及“Retype UNIX
    password:”，而使用该参数可以自定义输入密码的提示符，比如指定type=your own word；
    retry=N：定义登录/修改密码失败时，可以重试的次数；
    Difok=N：定义新密码中必须有几个字符要与旧密码不同。但是如果新密码中有1/2以上的字符与旧密码不同时，该新密码将被接受；
    minlen=N：定义用户密码的最小长度；
    dcredit=N：定义用户密码中必须包含多少个数字；
    ucredit=N：定义用户密码中必须包含多少个大写字母；
    lcredit=N：定义用户密码中必须包含多少个小些字母；
    ocredit=N：定义用户密码中必须包含多少个特殊字符（除数字、字母之外）；
    ```


