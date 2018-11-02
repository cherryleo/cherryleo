---
weight: 1000
title: "OpenLDAP是什么"
date: "2018-10-22"
lastmod: "2018-10-22"
description:  "openldap是什么，如何部署openldap，有哪些依赖，开放哪些端口，简单使用openldap"
categories:  ["devops"]
tags: ["openldap"]
---

## 1. 简介
LDAP（Lightweight Directory Access Protocol）代表轻量级目录访问协议。

LDAP通常用于访问目录服务，特别是基于X.500的目录服务。这里的目录指的是一种数据库，专门用于搜索和浏览的专用数据库，支持基本的查找和更新功能。
目录中存储的数据是条目，每一个条目就是一个DN（Distinguished Name）。

通常大部分工具软件都是支持LDAP的，比如Gitlab，Jenkins，Confluence等。而LDAP协议的好处就是公司的所有员工在所有这些工具里共享同一套用户名
和密码，新员工入职的时候在LDAP新增一个用户就能自动访问所有系统，离职的时候一键删除就取消了他对所有系统的访问权限。

基于域名的LDAP目录树，下图定义了example.com公司People部门的员工babs

![ldap-directory-tree.png](/openldap/ldap-directory-tree.png)

x.500协议属性

![x500-attributetype.png](/openldap/x500-attributetype.png)

## 2. 端口
389，非安全端口，ldap:///

636，安全端口，ldaps:///

## 3. 依赖
无

## 4. 安装
使用docker本地安装验证，启动openldap容器
```
docker run -d --name openldap \
  -p 389:389 \
  -e LDAP_ORGANISATION="My Company" \
  -e LDAP_DOMAIN="example.com" \
  -e LDAP_ADMIN_PASSWORD="123456" \
  osixia/openldap:1.2.2
```
启动phpldapadmin容器，该容器是ldap的web管理端，需要修改ldap的域名或者IP地址
```
docker run -d --name phpldapadmin \ 
  -p 6443:443 \
  -e PHPLDAPADMIN_LDAP_HOSTS=$your_ldap_ip \
  osixia/phpldapadmin:0.7.2
```

## 5. 验证
![php-ldapadmin-1.png](/openldap/phpldapadmin-1.png)

![php-ldapadmin-2.png](/openldap/phpldapadmin-2.png)