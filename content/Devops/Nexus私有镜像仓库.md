---
weight: 720
title: "Nexus私有镜像仓库"
date: "2018-11-07"
lastmod: "2018-11-07"
description:  "nexus创建docker私有镜像仓库，设置镜像仓库pull和push权限"
categories:  ["devops"]
tags: ["nexus"]
---

## 1. 私有镜像仓库
Nexus3版本可以作为docker镜像仓库，仓库分为三种类型，hosted为本地存储的私有仓库，proxy可以作为dockerhub的代理，
group可以聚合hosted和proxy仓库并提供统一的外部访问入口。

![registry.png](/nexus/registry.png)

## 2. 创建hosted类型镜像仓库
创建私有镜像仓库，需要强制勾选基础认证，否则docker login会出现401问题。如果nexus是使用docker启动，需要把镜像
仓库的端口映射出来。

![create-registry.png](/nexus/create-registry.png)

## 3. 使用私有镜像仓库
首先需要配置docker的insecure-registry，并重启docker。

![insecure-registry.png](/nexus/insecure-registry.png)

使用nexus用户登陆，admin/admin123

![login.png](/nexus/login.png)

tag镜像并push到nexus私有镜像仓库

![push.png](/nexus/push.png)

查看nexus上的镜像

![images.png](/nexus/images.png)

