---
weight: 810
title: "Jenkins对接LDAP"
date: "2018-11-09"
lastmod: "2018-11-09"
description:  "ldap用户登陆jenkins，jenkins对接ldap配置"
categories:  ["devops"]
tags: ["jenkins"]
---
## 1. LDAP用户
LDAP设置jenkins组和，jenkins组内有两位成员，张飞和关羽。

![ldap.png](/jenkins/ldap-jenkins.png)

## 2. 插件下载
Jenkins对接LDAP，需要下载LDAP插件。Jenkins插件支持在线安装和离线安装两种方式

在线安装方式：系统管理-->插件管理→可选插件-->查询ldap插件并安装

离线安装方式：下载插件到本地，系统管理-->插件管理-->高级→上传插件

## 3. 查看已安装插件
![ldap-plugin.png](/jenkins/ldap-plugin.png)

## 4. 对接配置
系统管理→全局安全配置，设置LDAP服务器连接信息，rootDN等信息，值得注意的是User search filter这个地方，这行进行LDAP
用户筛选，使用(&(cn={0})(memberOf=cn=jenkins,ou=groups,dc=example,dc=com))使只有jenkins组用户可以登录

![ldap-config-1.png](/jenkins/ldap-plugin.png)

设置LDAP只读用户readonly作为连接用户，测试LDAP连接信息

![ldap-config-2.png](/jenkins/ldap-plugin.png)
