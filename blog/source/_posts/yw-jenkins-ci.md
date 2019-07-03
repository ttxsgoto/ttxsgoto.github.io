---
title: Jenkins+Gitlab+Harbor构建持续集成
date: 2019-01-04 20:41:44
tags:
  - Jenkins
categories:
  - 运维
---

#### 背景
最近研究了一下公司现有的发布流程，大致流程是这样：开发在开发完成代码后，提PR后，Merge代码， 不需要关注代码发布流程， 使用Jenkins工具，自动发布代码到对应环境中，然后进行测试，提高研发工作效率和减少运维人员重复工作


#### 组件说明
##### Jenkins
jenkins是基于Java开发的一种持续集成工具，用于监控持续重复的工作，旨在提供一个开放易用的软件平台，使软件的持续集成变成可能
特点：
- 分布式构建
- 大量三方插件
- 变更支持
- 消息通知

##### Gitlab
Gitlab 是一个用于仓库管理系统的开源项目，使用Git作为代码管理工具，并在此基础上搭建起来的web服务
特点：
- 代码仓库管理
- 多人协作
- 完善的用户、权限管理

##### Harbor
Harbor 是一个用于存储和分发 Docker 镜像的企业级 Registry 服务器
特点：
- 基于角色的访问控制
- 镜像复制
- 鉴权认证管理
- 用户管理，访问控制和活动审计
- RESTful API

#### 实现说明
1. 通过Jenkins 生成的secret token来关联gitlab Webhooks设置url 和token，当代码有更新时，自动触发构建
2. 将代码拉取到Jenkins运行节点中，通过基础镜像加dockerfile文件构建应用镜像，然后上传到Harbor服务器中
3. 在测试服务器中拉取对应的镜像，然后运行起来
4. 构建完成没有错误，发邮件通知相关人员

#### 注意
1. Jenkins系统设置中需要配置gitlab服务器地址和对应的gitlab api token用于Jenkins和gitlab交互通信
2. Jenkins任务构建可以指定在固定节点上构建， 构建节点和应用服务器之间必须有登录权限
3. Harbor镜像管理，需要有鉴权和认证设置







