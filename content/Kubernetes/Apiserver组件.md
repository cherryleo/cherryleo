---
weight: 1310
title: "Apiserver组件"
date: "2018-11-12"
lastmod: "2018-11-12"
description:  "Apiserver组件，常用启动参数"
categories:  ["kubernetes"]
tags: ["kubernetes"]
---

## 简介

Apiserver组件是Kubernetes集群核心组件，所有其他组件均需要与apiserver进行交互。apiserver提供rest服务，并对k8s资源进行操作。

通常，与k8s集群进行交互即是与apiserver进行交互。apiserver也为集群提供授权和认证功能。

## 常用参数

apiserver支持超过100个参数，常用参数用来配置集群的认证模式，绑定的ip地址和端口，客户端证书，etcd地址等。

通过apiserver启动参数指定k8s集群提供的service地址范围，nodeport端口范围。

## 运行实例

启动apiserver，必须要提供etcd地址，如果没有etcd，则apiserver无法启动

```bash
$ ./kube-apiserver \
--etcd-servers=http://127.0.0.1:2379 \
--authorization-mode=Node,RBAC \
--advertise-address=10.255.0.168 \
--secure-port=6443 \
--insecure-port=0 \
--service-cluster-ip-range=10.96.0.0/12 \
--allow-privileged=true \
--client-ca-file=/etc/kubernetes/pki/ca.crt \
--tls-private-key-file=/etc/kubernetes/pki/apiserver.key \
--tls-cert-file=/etc/kubernetes/pki/apiserver.crt \
--kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname  \
--admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota
```

## 常用启动参数

```shell
--etcd-servers                                          etcd服务端列表(scheme://ip:port,scheme://ip:port)
--etcd-cafile                                           etcd ca文件
--etcd-certfile                                         etcd ssl通信证书
--etcd-keyfile                                          etcl ssl通信key
--etcd-prefix           默认值: "/registry"              etcd数据存储路径
--bind-address          默认值: 0.0.0.0                  安全端口监听地址, 不设置则使用0.0.0.0监听所有端口
--advertise-address                                     apiserver在集群中的通信IP地址, 若不设置则使用--bind-address参数值, 若--bind-address参数为空, 使用主机默认接口ip
--secure-port           默认值: 6443                     HTTPS端口, 设置0不启用
--basic-auth-file                                       http基础认证文件
--cert-dir              默认值: "/var/run/kubernetes"    TLS证书文件夹路径 若已设置--tls-cert-file 和 --tls-private-key-file参数, 忽略该参数
--client-ca-file                                        客户端ca证书文件 对客户端证书的commonname校验
--admission-control-config-file                         准入控制配置文件
--target-ram-mb                                         apiserver内存限制值, MB
--tls-cert-file                                         tls证书文件
--tls-private-key-file                                  tls私有key文件
--token-auth-file                                       token认证文件, 安全端口使用token进行认证
--service-cluster-ip-range       默认值: 10.0.0.0/24     集群service地址范围, 不能和pod地址范围冲突
--service-node-port-range        默认值: 30000-32767     NodePort范围
--storage-backend                默认值: etcd3           数据存储'etcd3' 或 'etcd2'
--allow-privileged               默认值: false           容器特权模式
--anonymous-auth                 默认值: true            允许匿名请求(匿名用户名/组名 system:anonymous/system:unauthenticated)
--apiserver-count                默认值: 1               集群中apiserver的数量
--authorization-mode             默认值: "AlwaysAllow"   安全端口授权模式列表: AlwaysAllow,AlwaysDeny,ABAC,Webhook,RBAC,Node
--authorization-policy-file                             授权模式为--authorization-mode=ABAC的csv格式授权策略文件
--enable-swagger-ui                                     启用swagger-ui, 路径/swagger-ui
--endpoint-reconciler-type       默认值: "master-count"  值(master-count, lease, none)
--kubelet-certificate-authority                         客户端证书认证文件
--kubelet-client-certificate                            客户端tls证书文件
--kubelet-client-key                                    客户端tls key文件
--kubelet-https                  默认值: true            使用https与kubelet建立连接
--kubelet-timeout duration       默认值: 5s              kubelet操作超时时间
```

