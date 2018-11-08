---
weight: 900
title: "Gitlab"
date: "2018-11-08"
lastmod: "2018-11-08"
description:  "gitlab是什么，如何部署gitlab，有哪些依赖，开放哪些端口，简单使用gitlab"
categories:  ["devops"]
tags: ["gitlab"]
---

## 1. 简介
Git是当前业界主流的分布式版本控制系统，相比SVN这种集中式的版本控制系统，更适合分布式协作开发、特性分支驱动开发。

GitLab是一个用Ruby on Rails开发的开源应用程序，实现一个自托管的Git项目仓库，可以通过Web界面进行访问公开的或者私人的项目。
可以把Gitlab理解为一个开源的Github，可以在企业中使用Gitlab部署一个企业自己的“Github”。

Gitlab拥有与Github类似的功能，能够浏览源代码，管理缺陷与注释。可以管理团队对仓库的访问，它非常易于浏览已提交过的版本并提供
一个文件历史库。它还提供一个代码片段收集功能可以轻松实现代码复用，便于日后有需要的时候进行查找。

## 2. 端口
80，web页面端口

443，web页面端口

22，ssh端口，git使用

## 3. 依赖
gitlab需要依赖redis和postgresql。二进制安装可以使用Gitlab CE Omnibus package这个包，可以一站式的解决安装、配置、管理备份
等需求。

## 4. 安装
使用docker本地安装验证，启动gitlab容器，容器镜像集成了所有的依赖。
```bash
docker run --detach \
	--hostname gitlab.example.com \
	--publish 443:443 --publish 80:80 --publish 22:22 \
	--name gitlab \
	--restart always \
	gitlab/gitlab-ce:latest
```

## 5. 验证
第一次访问web页面，需要设置超级管理员密码，之后使用root用户和刚设置的密码进行登陆

![init-login.png](/gitlab/init-login.png)
