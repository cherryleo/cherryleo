---
weight: 710
title: "Nexus对接LDAP"
date: "2018-11-07"
lastmod: "2018-11-07"
description:  "nexus对接LDAP，分配用户角色与权限"
categories:  ["devops"]
tags: ["nexus"]
---

## 1. LDAP配置
LDAP设置nexus组和administartors组，nexus组存放普通用户，administartors组存放管理员用户。三个用户，刘备为管理员，
张飞和关于为普通用户。

![ldap.png](/nexus/ldap.png)

## 2. 连接配置
配置LDAP连接信息，LDAP服务器地址以及建立连接用户。使用readonly用户建立连接，对LDAP数据库只有读权限，可以读取所有用户。

![ldapconnect.png](/nexus/ldapconnect.png)

## 3. 用户组和用户映射
映射LDAP用户和用户组，通过memberof查询输油nexus组和administrators组的用户

```bash
|(memberof=cn=nexus,ou=groups,dc=example,dc=com)(memberof=cn= administrators,ou=groups,dc=example,dc=com)
```

![userandgroup.png](/nexus/userandgroup.png)

![user.png](/nexus/user.png)

## 4. 设置nexus角色
把LDAP组和nexus组建立映射关系，设置组成员的角色，并设置组权限

![role.png](/nexus/role.png)
