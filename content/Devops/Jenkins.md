---
weight: 800
title: "Jenkins"
date: "2018-11-08"
lastmod: "2018-11-08"
description:  "jenkins是什么，如何部署jenkins，有哪些依赖，开放哪些端口，简单使用jenkins"
categories:  ["devops"]
tags: ["jenkins"]
---

## 1. 简介
Jenkins是开源的Java项目，用于自动执行，构建，测试，交付或部署软件相关的各种任务。

例如当提交代码到代码仓库后，就可以自动触发Jenkins流水线，帮助我们自动编辑代码，自动测试代码，打包软件，发布
软件等流程。

## 2. 端口
8080，web页面端口

50000，集群部署通讯端口

## 3. 依赖
无

## 4. 安装
使用docker本地安装验证，启动jenkins容器，jenkinsci/blueocean是官方集成blueocean的镜像
```bash
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 jenkinsci/blueocean
```

## 5. 验证

![unlock.png](/jenkins/unlock.png)

第一步，查看jenkins容器启动日志，获取密码
```bash
docker logs jenkins
```

![log.png](/jenkins/log.png)

第二部，自动安装插件，失败仍可以继续，也可以选择不安装插件

![auto-install-plugin.png](/jenkins/auto-install-plugin.png)

## 6. 版本
Jenkins版本主要有独立版本和集成blueocean的版本，关于版本的选择，官方推荐是使用blueocean版本。意思就是说blueocean版本
的镜像已经把blueocean插件集成了，不需要自己再安装插件。

BlueOcean是一个用户友好的pipeline面板，可以在blueocean里方便的查看每次流水线运行的情况，分阶段查看流水线的日志，查看
制品等功能。

![blueocean-tab.png](/jenkins/blueocean-tab.png)

![blueocean-pipeline.png](/jenkins/blueocean-pipeline.png)
