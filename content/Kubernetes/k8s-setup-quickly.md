---
title: "K8S - 安装 - 一键快速安装"
date: 2018-10-09T11:39:43+08:00
weight: 110
---

# K8S - 安装 - 一键快速安装

## 1. 简介

一键安装kubernetes master节点脚本，基于kubeadm方式安装集群，仅供测试使用

默认安装v1.10版本，所有资源均托管于腾讯云，无需翻墙



## 2. 环境要求

CentOS7+，Ubuntu16.04系统，安装过程中会自动安装docker17.03



## 3. 安装

```bash
$ wget -O - https://raw.githubusercontent.com/cherryleo/cherryleo/master/install-k8s-cluster.sh |sudo bash
```



## 4. 插件安装

```bash
# 网络插件安装，此处flannel网络
$ kubectl apply -f https://raw.githubusercontent.com/cherryleo/k8s-apps/master/k8s-flannel/flannel.yaml
# dashboard安装
$ kubectl apply -f https://raw.githubusercontent.com/cherryleo/k8s-apps/master/k8s-dashboard/kubernetes-dashboard.yaml
# 创建admin用户
$ kubectl apply -f https://raw.githubusercontent.com/cherryleo/k8s-apps/master/k8s-dashboard/admin-user.yaml
```



## 5. 验证

```bash
$ kubectl get nodes
NAME          STATUS    ROLES     AGE       VERSION
10-255-0-41   Ready     master    2m        v1.10.0
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                    READY     STATUS    RESTARTS   AGE
kube-system   etcd-10-255-0-41                        1/1       Running   0          4m
kube-system   kube-apiserver-10-255-0-41              1/1       Running   1          5m
kube-system   kube-controller-manager-10-255-0-41     1/1       Running   0          5m
kube-system   kube-dns-7dd59b9bdb-nv7m9               3/3       Running   0          5m
kube-system   kube-flannel-ds-8vjl7                   1/1       Running   0          5m
kube-system   kube-proxy-6gjbl                        1/1       Running   0          5m
kube-system   kube-scheduler-10-255-0-41              1/1       Running   0          5m
kube-system   kubernetes-dashboard-6888bf8db6-8vsvt   1/1       Running   0          5m
```

