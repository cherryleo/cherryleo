# Prometheus告警配置

## 1. 配置alertmanager实例

alertmanager可以静态配置，也可通过服务发现机制动态发现


```yaml
# prometheus.yml
alerting:
  alertmanagers:
    # 向alertmanager推送告警时间间隔
    [ timeout: <duration> | default = 10s ]

    # Prefix for the HTTP path alerts are pushed to.
    [ path_prefix: <path> | default = / ]
    [ scheme: <scheme> | default = http ]

    # 安全认证相关
    basic_auth:
      [ username: <string> ]
      [ password: <string> ]
    [ bearer_token: <string> ]
    [ bearer_token_file: /path/to/bearer/token/file ]
    tls_config:
      [ <tls_config> ]

    [ proxy_url: <string> ]

    # alertmanager地址
    static_configs:
      [ - <static_config> ... ]

    # List of Alertmanager relabel configurations.
    relabel_configs:
      [ - <relabel_config> ... ]
```



## 2. 配置告警规则

