---
title: 运维 Hexo常用命令
date: 2017-02-01 22:38:32
tags:
  - Hexo
categories:
  - 运维
---
#### 安装命令
```
npm install -g hexo # 安装hexo
hexo init   #初始化hexo目录
npm install hexo-deployer-git --save 安装git所需的插件

```
#### 常用命令
```
hexo clean
hexo generate # 或者 hexo g 生成静态网页 
hexo server # 或者 hexo s 启动本地服务
hexo  deploy    #将本地文件推送到github上
hexo new "title"    # 新建文章
hexo new page "pagename"    # 新建页面

```

