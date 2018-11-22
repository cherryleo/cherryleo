---
weight: 600
title: "Nexus"
date: "2018-11-14"
lastmod: "2018-11-14"
description:  "nexus是什么，如何部署nexus，有哪些依赖，开放哪些端口，简单使用nexus"
categories:  ["devops"]
tags: ["prometheus"]
---


## 1. 简介
Prometheus 是由 SoundCloud 开发的开源监控报警系统和时序列数据库(TSDB)，Prometheus 在2016加入 CNCF ( Cloud Native Computing Foundation ), 作为在 kubernetes 之后的第二个由基金会主持的项目。

github地址（https://github.com/prometheus）

## 2. 端口
9090：prometheus server
9091：pushgateway
9092：未分配
9093：alertmanager
9094：alertmanager cluster

## 3. 依赖
无

## 4. 安装
使用docker本地安装验证，启动openldap容器
```bash
docker run -d --name openldap \
  -p 389:389 \
  -e LDAP_ORGANISATION="My Company" \
  -e LDAP_DOMAIN="example.com" \
  -e LDAP_ADMIN_PASSWORD="123456" \
  osixia/openldap:1.2.2
```
启动phpldapadmin容器，该容器是ldap的web管理端，需要修改ldap的域名或者IP地址
```bash
docker run -d --name phpldapadmin \ 
  -p 6443:443 \
  -e PHPLDAPADMIN_LDAP_HOSTS=$your_ldap_ip \
  osixia/phpldapadmin:0.7.2
```

## 5. 验证
![php-ldapadmin-1.png](/openldap/phpldapadmin-1.png)

![php-ldapadmin-2.png](/openldap/phpldapadmin-2.png)