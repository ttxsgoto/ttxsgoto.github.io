---
title: Apollo配置中心
date: 2019-01-07 20:59:45
tags:
  - Apollo
categories:
  - 运维
---
#### 背景
之前文章中说过发布流程([Jenkins+Gitlab+Harbor构建持续集成](https://ttxsgoto.github.io/2019/01/04/yw-jenkins-ci/)),但这里有个问题——不同环境中的配置文件不一样，之前的方案只是把代码发布到机器上，不能“一刀切”使用相同的配置，我们现在使用携程框架部门研发的开源配置管理中心Apollo来完成配置文件分发

#### 特点
- 配置修改实时生效
- 灰度发布
- 分环节
- 分集群管理配置
- 权限
- 审核机制等

apollo能够集中化管理应用不同环境，不同集群，修改配置后能够实时推送到应用，并具备规范的权限和流程治理特性

#### 4个维度管理k-v配置

- application(应用)
- environment(环境)
- cluster(集群)
- namespace(命名空间)

配置文件实时生效(热发布),用户在apollo修改完配置并发布后，客户端能实时(1s)接收到最新配置，并通知应用程序

#### 客户端实现
客户端(python)实现参照
https://github.com/filamoon/pyapollo

#### 参考链接
- https://github.com/ctripcorp/apollo
- https://github.com/ctripcorp/apollo/wiki/Apollo%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83%E4%BB%8B%E7%BB%8D
- https://github.com/ctripcorp/apollo/wiki/%E5%85%B6%E5%AE%83%E8%AF%AD%E8%A8%80%E5%AE%A2%E6%88%B7%E7%AB%AF%E6%8E%A5%E5%85%A5%E6%8C%87%E5%8D%97

