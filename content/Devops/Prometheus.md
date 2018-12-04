---
weight: 600
title: "Prometheus"
date: "2018-11-14"
lastmod: "2018-12-03"
description:  "prometheus简介，prometheus组件，启动参数与配置文件参数"
categories:  ["devops"]
tags: ["prometheus"]
---

## 1. 简介
Prometheus 是由 SoundCloud 开发的开源监控报警系统和时序列数据库(TSDB)，Prometheus 在2016加入 CNCF ( Cloud Native Computing Foundation ), 作为在 kubernetes 之后的第二个由基金会主持的项目。

github地址（https://github.com/prometheus）

## 2. 端口
__9090__：prometheus server

__9091__：pushgateway

__9092__：未分配

__9093__：alertmanager

__9094__：alertmanager cluster

## 3. 依赖
无

## 4. 组件
__prometheus server__：主要用于抓取数据和存储时序数据，另外还提供查询和 Alert Rule 配置管理

__client libraries__：用于对接 Prometheus Server, 可以查询和上报数据

__push gateway__：用于批量，短期的监控数据的汇总节点，主要用于业务数据汇报等

__exporters__：各种汇报exporter，例如node_exporter，mysql_exporter，mongodb_exporter

__alertmanager__：告警通知管理

## 5. 配置
prometheus使用启动参数和配置文件进行配置，启动参数配置系统参数，配置文件配置作业和实例内容

#### 启动参数
__--config.file__：配置文件绝对路径，默认值"prometheus.yml"

__--web.listen-address__：UI/API监听地址，默认值"0.0.0.0:9090"

__--web.max-connections__：最大连接数，默认值512

__--web.external-url__：外部访问prometheus地址

__--web.enable-admin-api__：启用管理端api

__--web.enable-lifecycle__：可以通过HTTP重启或者关闭服务

__--web.console.templates__：绝对路径，默认值"consoles" 

__--web.console.libraries__：绝对路径，默认值"console_libraries" 

__--storage.tsdb.path__：绝对路径，默认值"data/" 

__--storage.tsdb.retention__：数据样本存储时间，默认值15d

#### 配置文件
```yaml
# Prometheus 启动的时候，可以加载运行参数 -config.file 指定配置文件，默认为 prometheus.yml
# 全局配置
global:
  scrape_interval: 15s # 拉取 targets 的默认时间间隔
  scrape_timeout: 15s  # 拉取一个 target 的超时时间
  evaluation_interval: 15s  # 执行 rules 的时间间隔
  # 额外的属性，会添加到拉取的数据并存到数据库中
  external_labels:
    monitor: 'codelab-monitor'

# 告警配置
alerting:
  # 动态修改 alert 属性的规则配置
  alert_relabel_configs:
    [ - <relabel_config> ... ]
  # 动态发现 Alertmanager 的配置
  alertmanagers:
    [ - <alertmanager_config> ... ]

# 规则配置
rule_files:
  - "rules/node.rules"
  - "rules2/*.rules"

# 数据拉取配置
scrape_configs:
  - job_name: 'prometheus'  # 任务名称
    scrape_interval: 5s     # 拉取时间间隔
    # scrape_timeout: 拉取超时时间
    metrics_path：/prometheus/metrics  # 拉取节点的 metric 路径
    # scheme： 拉取数据访问协议
    # 静态服务发现
    static_configs:
      - targets: ['localhost:9090']
```