---
weight: 700
title: "Nexus"
date: "2018-11-07"
lastmod: "2018-11-07"
description:  "nexus是什么，如何部署nexus，有哪些依赖，开放哪些端口，简单使用nexus"
categories:  ["devops"]
tags: ["nexus"]
---

## 1.简介
Nexus是一个存储库管理器，用于存储开发使用的软件包和软件制品。

软件开发过程中通常需要从外部下载依赖，比如python使用pip下载依赖包，java通过maven从中央仓库拉取依赖。在使用nexus之后
可以统一从nexus下载依赖，开发完成后可以把自己的软件制品发布到nexus共享给他人。

Nexus还可以作为docker私有镜像仓库，yum源仓库。

## 2. 端口
8081，默认web页面端口

## 3. 依赖
无

## 4. 安装
使用容器在本地安装验证，安装nexus3版本

```bash
docker run -d -p 8081:8081 --name nexus sonatype/nexus3
```

## 5. 验证
默认用户名密码admin/admin123

![nexus-firstpage.png](/nexus/nexus-firstpage.png)