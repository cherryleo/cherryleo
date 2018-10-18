# Alertmanager

## 1. 简介

Alertmanager处理客户端应用程序（如Prometheus服务器）发送的警报。它负责重复数据删除，分组并将它们路由到正确的接收方集成，例如电子邮件，PagerDuty或OpsGenie。它还处理警报的沉默和抑制。



## 2. 安装

### 2.1 依赖

无



### 2.2 二进制

```
$ ./alertmanager --config.file=<your_file>
```



### 2.3 容器

[镜像](https://quay.io/repository/prometheus/alertmanager)



## 3. 配置 

### 3.1 命令行参数

| 参数                                                  | 说明         |
| ----------------------------------------------------- | ------------ |
| --config.file="alertmanager.yml"                      | 配置文件路径 |
| --storage.path="data/"                                | 存储路径     |
| --data.retention=120h                                 | 存储时效     |
| --web.external-url=WEB.EXTERNAL-URL                   | 外部url      |
| --web.listen-address=":9093"                          | 端口         |
| --cluster.listen-address="0.0.0.0:9094"               | 集群端口     |
| --cluster.advertise-address=CLUSTER.ADVERTISE-ADDRESS | 集群外部地址 |



### 3.2 配置文件

#### 3.2.1 全局配置

```yaml
global:
  [ resolve_timeout: <duration> | default = 5m ]

  # smtp配置
  [ smtp_from: <tmpl_string> ]
  [ smtp_smarthost: <string> ]
  [ smtp_hello: <string> | default = "localhost" ]
  [ smtp_auth_username: <string> ]
  [ smtp_auth_password: <secret> ]
  [ smtp_auth_identity: <string> ]
  [ smtp_auth_secret: <secret> ]
  [ smtp_require_tls: <bool> | default = true ]

  # The API URL to use for Slack notifications.
  [ slack_api_url: <string> ]
  [ wechat_api_url: <string> | default = "https://qyapi.weixin.qq.com/cgi-bin/" ]
  [ wechat_api_secret: <secret> ]
  [ wechat_api_corp_id: <string> ]

  # The default HTTP client configuration
  [ http_config: <http_config> ]

# 通知模版路径, example: - '/etc/alertmanager/template/*.tmpl'
templates:
  [ - <filepath> ... ]

# 路由
route: <route>

# 告警接受者列表
receivers:
  - <receiver> ...

# 禁止规则列表
inhibit_rules:
  [ - <inhibit_rule> ... ]
```



#### 3.2.2 路由配置

```yaml
route:
  [ receiver: <string> ]
  # 根据标签进行告警分组
  [ group_by: '[' <labelname>, ... ']' ]

  # 告警规则匹配，false在第一次匹配成功后停止，true继续匹配规则
  [ continue: <boolean> | default = false ]

  # 告警规则，全词匹配
  match:
    [ <labelname>: <labelvalue>, ... ]
  # 告警规则，正则匹配
  match_re:
    [ <labelname>: <regex>, ... ]

  # 产生告警到发送告警的时间间隔
  [ group_wait: <duration> | default = 30s ]
  # 发送告警后，再次发送新告警时间间隔
  [ group_interval: <duration> | default = 5m ]
  # 同一个告警再次发送时间间隔
  [ repeat_interval: <duration> | default = 4h ]

  # 子路由
  routes:
    [ - <route> ... ]
```



#### 3.2.3 抑制规则

```yaml
inhibit_rules:
  # Matchers that have to be fulfilled in the alerts to be muted.
  - target_match:
      [ <labelname>: <labelvalue>, ... ]
    target_match_re:
      [ <labelname>: <regex>, ... ]

    # Matchers for which one or more alerts have to exist for the
    # inhibition to take effect.
    source_match:
      [ <labelname>: <labelvalue>, ... ]
    source_match_re:
      [ <labelname>: <regex>, ... ]

    # Labels that must have an equal value in the source and target
    # alert for the inhibition to take effect.
    [ equal: '[' <labelname>, ... ']' ]
```



#### 3.2.4 HTTP配置

```yaml
http_config:
  basic_auth:
    [ username: <string> ]
    [ password: <secret> ]

  [ bearer_token: <secret> ]
  [ bearer_token_file: <filepath> ]

  tls_config:
    [ ca_file: <filepath> ]
    [ cert_file: <filepath> ]
    [ key_file: <filepath> ]
    [ server_name: <string> ]
    [ insecure_skip_verify: <boolean> | default = false]

  [ proxy_url: <string> ]
```



#### 3.2.5 Receiver配置

```yaml
receivers:
  - name: <string>
    # 配置多种告警集成
    email_configs:
      [ - <email_config>, ... ]
    webhook_configs:
      [ - <webhook_config>, ... ]
    wechat_configs:
      [ - <wechat_config>, ... ]
```



#### 3.2.6 邮件配置

```yaml
email_configs:
  # 是否通知已解决的告警
  - [ send_resolved: <boolean> | default = false ]
    
    # 邮件收发地址
    to: <tmpl_string>
    [ from: <tmpl_string> | default = global.smtp_from ]

    # smtp配置
    [ smarthost: <string> | default = global.smtp_smarthost ]
    [ hello: <string> | default = global.smtp_hello ]
    [ auth_username: <string> | default = global.smtp_auth_username ]
    [ auth_password: <secret> | default = global.smtp_auth_password ]
    [ auth_secret: <secret> | default = global.smtp_auth_secret ]
    [ auth_identity: <string> | default = global.smtp_auth_identity ]
    [ require_tls: <bool> | default = global.smtp_require_tls ]

    # 邮件告警正文
    [ html: <tmpl_string> | default = '{{ template "email.default.html" . }}' ]
    [ text: <tmpl_string> ]

    # 邮件头
    [ headers: { <string>: <tmpl_string>, ... } ]
```



#### 3.2.7 微信配置

```yaml
wechat_configs:
  # 是否通知已解决的告警
  - [ send_resolved: <boolean> | default = false ]

    # 微信api配置
    [ api_secret: <secret> | default = global.wechat_secret_url ]
    [ api_url: <string> | default = global.wechat_api_url ]
    [ corp_id: <string> | default = global.wechat_api_corp_id ]

    # 告警内容与收发者配置
    [ message: <tmpl_string> | default = '{{ template "wechat.default.message" . }}' ]
    [ agent_id: <string> | default = '{{ template "wechat.default.agent_id" . }}' ]
    [ to_user: <string> | default = '{{ template "wechat.default.to_user" . }}' ]
    [ to_party: <string> | default = '{{ template "wechat.default.to_party" . }}' ]
    [ to_tag: <string> | default = '{{ template "wechat.default.to_tag" . }}' ]
```



#### 3.2.8 配置示例

```yaml
global:
  # The smarthost and SMTP sender used for mail notifications.
  smtp_smarthost: 'localhost:25'
  smtp_from: 'alertmanager@example.org'
  smtp_auth_username: 'alertmanager'
  smtp_auth_password: 'password'

# The directory from which notification templates are read.
templates: 
- '/etc/alertmanager/template/*.tmpl'

# The root route on which each incoming alert enters.
route:
  # The labels by which incoming alerts are grouped together. For example,
  # multiple alerts coming in for cluster=A and alertname=LatencyHigh would
  # be batched into a single group.
  group_by: ['alertname', 'cluster', 'service']

  # When a new group of alerts is created by an incoming alert, wait at
  # least 'group_wait' to send the initial notification.
  # This way ensures that you get multiple alerts for the same group that start
  # firing shortly after another are batched together on the first 
  # notification.
  group_wait: 30s

  # When the first notification was sent, wait 'group_interval' to send a batch
  # of new alerts that started firing for that group.
  group_interval: 5m

  # If an alert has successfully been sent, wait 'repeat_interval' to
  # resend them.
  repeat_interval: 3h 

  # A default receiver
  receiver: team-X-mails

  # All the above attributes are inherited by all child routes and can 
  # overwritten on each.

  # The child route trees.
  routes:
  # This routes performs a regular expression match on alert labels to
  # catch alerts that are related to a list of services.
  - match_re:
      service: ^(foo1|foo2|baz)$
    receiver: team-X-mails
    # The service has a sub-route for critical alerts, any alerts
    # that do not match, i.e. severity != critical, fall-back to the
    # parent node and are sent to 'team-X-mails'
    routes:
    - match:
        severity: critical
      receiver: team-X-pager
  - match:
      service: files
    receiver: team-Y-mails

    routes:
    - match:
        severity: critical
      receiver: team-Y-pager

  # This route handles all alerts coming from a database service. If there's
  # no team to handle it, it defaults to the DB team.
  - match:
      service: database
    receiver: team-DB-pager
    # Also group alerts by affected database.
    group_by: [alertname, cluster, database]
    routes:
    - match:
        owner: team-X
      receiver: team-X-pager
    - match:
        owner: team-Y
      receiver: team-Y-pager


# Inhibition rules allow to mute a set of alerts given that another alert is
# firing.
# We use this to mute any warning-level notifications if the same alert is 
# already critical.
inhibit_rules:
- source_match:
    severity: 'critical'
  target_match:
    severity: 'warning'
  # Apply inhibition if the alertname is the same.
  equal: ['alertname', 'cluster', 'service']


receivers:
- name: 'team-X-mails'
  email_configs:
  - to: 'team-X+alerts@example.org'

- name: 'team-X-pager'
  email_configs:
  - to: 'team-X+alerts-critical@example.org'

- name: 'team-Y-mails'
  email_configs:
  - to: 'team-Y+alerts@example.org'
```