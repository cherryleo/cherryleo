---
weight: 1010
title: "OpenLDAP简单使用"
date: "2018-10-22"
lastmod: "2018-10-22"
description:  "openldap如何配置，如何连接ldap服务器，如何添加用户、用户组"
categories:  ["devops"]
tags: ["openldap"]
---

## 1. 场景
example.com公司有两个组织单元，people和groups。people目录下存储所有员工条目，groups目录下存储所有组条目。

people目录下有三个用户，刘备、关羽、张飞，groups目录下有两个组，gitlab组，jenkins组。

用户刘备属于gitlab和jenkins组，用户关羽只属于gitlab组，用户张飞只属于jenkins组。

## 2. 创建OpenLDAP实例
使用docker创建ldap服务实例，默认超级用户admin，密码123456
```
docker run -d --name my-openldap -p 389:389 \
  -e LDAP_ORGANISATION="Example Inc" \
  -e LDAP_READONLY_USER=true \
  -e LDAP_DOMAIN="example.com" \
  -e LDAP_ADMIN_PASSWORD="123456" osixia/openldap:1.2.2
```

## 3. 创建用户和组
把用户和组数据按照ldif格式保存为ldif文件（类似把sql语句保存为sql文件，并load进数据库），使用ldapadd工具把数据导入ldap。

![ldapadd.png](/openldap/ldapadd.png)

```bash
# data.ldif
# ou, 两个组织单元people和groups
dn: ou=groups,dc=example,dc=com
objectclass: organizationalUnit
ou: groups

dn: ou=people,dc=example,dc=com
objectclass: organizationalUnit
ou: people

# people, people目录员工条目
dn: cn=liubei,ou=people,dc=example,dc=com
objectclass: inetOrgPerson
cn: liubei
displayName: 刘备    
givenName: bei
sn: liu
mail: bei.liu@example.com
userpassword: {MD5}4QrcOUm6Wau+VuBX8g+IPg==

dn: cn=guanyu,ou=people,dc=example,dc=com
objectclass: inetOrgPerson
cn: guanyu
displayName: 关羽    
givenName: yu
sn: guan
mail: yu.guan@example.com
userpassword: {MD5}4QrcOUm6Wau+VuBX8g+IPg==

dn: cn=zhangfei,ou=people,dc=example,dc=com
objectclass: inetOrgPerson
cn: zhangfei
displayName: 张飞    
givenName: fei
sn: zhang
mail: fei.zhang@example.com
userpassword: {MD5}4QrcOUm6Wau+VuBX8g+IPg==

# group, 员工于所属组关联
dn: cn=gitlab,ou=groups,dc=example,dc=com
objectclass: groupOfUniqueNames
cn: gitlab
uniqueMember: cn=liubei,ou=people,dc=example,dc=com
uniqueMember: cn=guanyu,ou=people,dc=example,dc=com

dn: cn=jenkins,ou=groups,dc=example,dc=com
objectclass: groupOfUniqueNames
cn: jenkins
uniqueMember: cn=liubei,ou=people,dc=example,dc=com
uniqueMember: cn=zhangfei,ou=people,dc=example,dc=com
```

## 4. 查询条目验证
使用ldapsearch进行查询，下图查询ldap中存储的所有数据

![ldapsearch.png](/openldap/ldapsearch.png)

查看组和用户的关联情况

![ldapmember-1.png](/openldap/ldapmember-1.png)

![ldapmember-2.png](/openldap/ldapmember-2.png)
