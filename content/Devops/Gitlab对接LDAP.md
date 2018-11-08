---
weight: 910
title: "Gitlab对接LDAP"
date: "2018-11-08"
lastmod: "2018-11-08"
description:  "gitlab用户设置，LDAP用户直接登陆gitlab"
categories:  ["devops"]
tags: ["gitlab"]
---

## 1. LDAP用户
LDAP设置gitlab组和，gitlab组内有两位成员，张飞和关羽。

![ldap.png](/gitlab/ldap.png)

## 2. Gitlab配置
修改/etc/gitlab/gitlab.rb文件，设置ldap服务器地址和端口，并过滤出gitlab组用户。映射ldap属性。

```bash
# LDAP
gitlab_rails['ldap_enabled'] = true
gitlab_rails['ldap_servers'] = YAML.load <<-'EOS'
main: # 'main' is the GitLab 'provider ID' of this LDAP server
    label: 'LDAP'
    host: 'ldap-server'
    port: 389
    uid: 'cn'
    method: 'plain' # "tls" or "ssl" or "plain"
    bind_dn: 'cn=readonly,dc=example,dc=com'
    password: '123456'
    active_directory: false
    allow_username_or_email_login: true
    block_auto_created_users: false
    base: 'ou=people,dc=example,dc=com'
    user_filter: '(memberof=cn=gitlab,ou=groups,dc=example,dc=com)'
    attributes:
    username: ['cn']
    email:    ['mail']
    name:       'displayName'
    first_name: 'givenName'
    last_name:  'sn'
EOS
```

执行gitlab-ctl reconfigure使配置生效

![reconfigure.png](/gitlab/reconfigure.png)

## 3. 验证
![ldap-status.png](/gitlab/ldap-status.png)

![dap-user.png](/gitlab/ldap-user.png)

