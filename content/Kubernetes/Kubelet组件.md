---
weight: 1300
title: "Kubelet组件"
date: "2018-11-12"
lastmod: "2018-11-12"
description:  "Kubelet组件，常用启动参数"
categories:  ["kubernetes"]
tags: ["kubernetes"]
---

## 简介

Kubelet作为节点代理在每一个节点上运行，主要工作就是创建和管理节点上的Pod（k8s容器组）。通常我们使用一个YAML或者JSON对象描述Pod，Kubelet接收YAML或者JSON数据对Pod进行创建和管理。

Kubelet主要接收来自apiserver组件的数据，可以在启动kubelet时指定主机某个路径下的Pod描述文件，也可以监听某个网络位置上的Pod描述文件进行Pod创建和管理。

## 常用参数

Kubelet支持100多个参数，包括一些特性功能，实验功能等。Kubelet通常作为守护进程或者系统服务运行在节点上。

启动kubelet常用参数是指定集群的配置（如集群域名，集群dns地址，集群pod网络地址等），节点配置（主机名，ip地址等），kubelet配置（数据目录，如何与集群apiserver通信，需要静态启动的pod等）。

## 运行实例

```shell
$ ./kubelet \
--kubeconfig=/etc/kubernetes/kubelet.conf \
--pod-manifest-path=/etc/kubernetes/manifests \
--pod-infra-container-image=my.registry.com/k8s/pause-amd64:3.1 \
--allow-privileged=true \
--network-plugin=cni \
--cni-conf-dir=/etc/cni/net.d \
--cni-bin-dir=/opt/cni/bin \
--cluster-dns=10.96.0.1 \
--cluster-domain=cluster.local \
--cgroup-driver=systemd \
--fail-swap-on=false
```

## 部分常用启动参数

```bash
--address                  默认值: 0.0.0.0            kubelet监听地址, 0.0.0.0监听所有接口
--port                     默认值: 10250              kubelet服务端口
--read-only-port           默认值: 10255              kubelet只读端口,针对匿名请求, 设置 0 不启用
--healthz-bind-address     默认值: 127.0.0.1          健康检查监听地址, 0.0.0.0监听所有端口
--healthz-port             默认值: 10248              健康检查监听端口, 设置0不启用
--hostname-override                                  设置hostname, 设置后k8s会使用设置名作为主机名
--container-runtime        默认值: "docker"           容器运行环境, "docker", "rkt"
--enable-server            默认值: true               启用kubelet server
--cluster-dns                                        集群dns地址, ip地址列表
--cluster-domain                                     集群域名
--node-ip                                            节点ip地址, 设置后kubelet使用该ip作为节点地址
--cni-bin-dir              默认值: /opt/cni/bin       cni插件路径
--cni-conf-dir             默认值: /etc/cni/net.d     cni配置文件路径
--network-plugin                                     网络插件名称
--pod-cidr                                           pod ip地址范围
--max-open-files           默认值: 1000000            kubelet进程打开文件数量最大值
--max-pods                 默认值: 110                kubelet运行pod最大数量
--allow-privileged                                   运行容器请求特权模式, true 或 false 
--pod-manifest-path                                  pod描述文件路径, kubelet会根据该路径下的pod描述文件创建pod
--config                                             配置文件, 相对路径或绝对路径, 忽略该参数则使用内置默认值
--register-node            默认值: true Register      自动向apiserver注册
--root-dir                 默认值: "/var/lib/kubelet" kubelet根路径
--anonymous-auth           默认值: true               允许匿名访问kubelet, 匿名用户名/组名(system:anonymous/system:unauthenticated)
--cadvisor-port            默认值: 4194                           本地cAdvisor端口, 设置 0 不启用cAdvisor
--cert-dir                 默认值: "/var/lib/kubelet/pki"         TLS证书路径, 如果设置 --tls-cert-file 和 --tls-private-key-file参数, 该参数会被忽略
--tls-cert-file                                                  x509证书文件
--tls-private-key-file                                           x509私有密钥
--kubeconfig               默认值: "/var/lib/kubelet/kubeconfig"  连接apiserver配置文件路径
--cgroup-driver            默认值: "cgroupfs"                     cgroup驱动, "cgroupfs" 或 "systemd"
--authorization-mode       默认值: "AlwaysAllow"                  授权模式, AlwaysAllow 或 Webhook
--docker-endpoint          默认值: "unix:///var/run/docker.sock"  docker endpoint
--fail-swap-on             默认值: true                           如果swap开启, kubelet会启动失败
--image-gc-high-threshold       默认值: 85            磁盘空间百分比, 磁盘空间超过该比例进行镜像垃圾回收
--image-gc-low-threshold        默认值: 80            磁盘空间百分比, 磁盘空间没超过该比例之前不会进行垃圾回收
--image-pull-progress-deadline  默认值: 1m0s          在设置的时间内无法拉取镜像, 则取消镜像拉取
--minimum-image-ttl-duration    默认值: 2m0s          被垃圾回收镜像最小寿命
--pod-infra-container-image     默认值: "gcr.io/google_containers/pause-amd64:3.1"   pause镜像名称
--container-runtime-endpoint    默认值: "unix:///var/run/dockershim.sock"            可配置远程服务(实验特性)
--containerized                                                                     以容器方式运行kubelet(实验特性)
```



