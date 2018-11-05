---
weight: 1020
title: "OpenLDAP权限"
date: "2018-10-22"
lastmod: "2018-10-23"
description:  "openldap权限配置，用户权限分配，用户密码管理"
categories:  ["devops"]
tags: ["openldap"]
---

## 1. 目的
在使用LDAP管理用户，必然会出现权限问题。首先需要有一个管理员能够对所有用户信息进行读写，然后用户自己可以修改自己的密码。
如果有组的话，还需要设定组管理员，只读用户等。这些权限的配置均通过访问控制表实现。

比如gitlab，jenkins等系统与LDAP对接时，可以分配一个全局只读用户用于LDAP数据读取。

## 2. 使用场景
公司是example.com，下属两个组织单元people和groups以及admin和readonly用户。

people目录存放的是公司所有员工条目，groups目录存放用户组信息。

通过制定权限策略，使admin用户拥有全局读写权限，readonly用户拥有全局读权限，people目录下的用户对people目录具有读权限，
普通用户对自己的密码具有写权限。

## 3. acl配置
关于ldap的权限配置，可以在启动时通过配置文件指定，也可以在使用ldapmodify工具动态修改配置

acl的基本语法模式就是一个to后面跟多个by，to 属性 by 谁 权限 by 谁 权限 by ...，从第一条策略开始匹配，遇到第一个匹配的策略停止。

首先查看当前ldap的数据库配置，当前ldap服务器使用的数据库是{1}mdb

![searchdb.png](/openldap/searchdb.png)

查看{1}mdb数据库的acl配置，当前两条规则确定了admin为超级管理员，全局可读写，所有用户自己可以修改自己的密码

![searchacl.png](/openldap/searchacl.png)

当使用phpldapadmin登陆时，普通用户没有读权限，所以尽管可以修改自己的密码，但无法读密码则造成无法修改密码。需要
新增acl策略，增加一个所有数据普通用户可读，否则phpldapadmin登陆无法读取信息，也就无法更改自己的密码。更好的方
式是能够提供自服务式的密码管理，仅供用户修改密码。

```
# acl.ldif
dn: olcDatabase={1}mdb,cn=config
changetype: modify
add: olcAccess
olcAccess: {1}to * by dn="cn=admin,dc=example,dc=com" write by users read by anonymous auth
```

![searchacl-2.png](/openldap/searchacl-2.png)

## 4. 验证
此处使用phpldapadmin进行验证，普通用户修改除了password字段，都是无权限的

![phpldaplogin.png](/openldap/phpldaplogin.png)


## 5. 密码自服务管理
通常来说，ldap的用户是拥有修改自己密码的权限（管理员新分配的ldap用户，需要用户修改自己的密码）。需要提供一个自服务式
的密码管理工具，让用户仅仅修改自己的密码。

小工具地址，https://github.com/cherryleo/ldappasswd

![selfpasswd.png](/openldap/selfpasswd.png)
