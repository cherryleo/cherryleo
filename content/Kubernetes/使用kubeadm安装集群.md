---
weight: 1100
title: "使用Kubeadm安装集群"
date: "2018-11-12"
lastmod: "2018-11-12"
description:  "Kubernetes集群安装，使用Kubeadm安装Kubernetes集群"
categories:  ["kubernetes"]
tags: ["kubernetes"]
---

## 简介

Kubeadm是kubernetes官方工具，用来快速安装kubernetes集群。整套工具由kubeadm，kubelet，kubernetes-cni和kubectl包组成，
除kubelet以二进制方式运行外，其余组件均作为kubernetes静态pod启动（镜像均托管谷歌与gcr.io仓库）。

目前kubeadm支持通过配置文件imageRepository指定镜像仓库地址，个人在使用和学习过程中同步一些官方镜像于腾讯云镜像仓库，并写了
一些自动部署的脚本，以便快速部署kubernetes集群。

本文档详细介绍使用kubadm部署k8s集群完整步骤

## 依赖配置

系统发行版CentOS 7+，amd64，master节点配置2核2G以上，需要安装以下软件包

```bash
# CentOS7
$ sudo yum install ebtables ethtool iproute iptables socat util-linux wget -y
```

安装docker，使用脚本自动安装docker-ce-17，已有docker可跳过此步

```bash
# CentOS7安装docker-ce-17.03
$ sudo wget -O - https://raw.githubusercontent.com/cherryleo/scripts/master/centos7-install-docker.sh | sudo sh
```

系统设置，设置环境变量指定安装Kubernetes版本，默认1.10版本

```bash
# 关闭swap
$ sudo swapoff -a

# 关闭防火墙，如果不关防火墙，确保8080，6443，10250端口开放
$ sudo systemctl disable firewalld
$ sudo systemctl stop firewalld

# 修改网络参数
$ sudo sysctl net.bridge.bridge-nf-call-iptables=1

# 设置环境变量，k8s安装版本，支持版本1.9.0 -- 1.9.9, 1.10.0 -- 1.10.5
$ export KUBERNETES_VERSION="1.10.0"
```

脚本自动安装kubeadm，kubeadm的rpm包同步于腾讯云

```bash
$ wget -O - https://raw.githubusercontent.com/cherryleo/scripts/master/install-k8s-packages.sh | sudo -E bash
```

## Kubeadm配置

查看docker cgroup driver

```bash
$ sudo docker info | grep -i cgroup
```

修改kubeadm配置文件，在配置文件中指定docker cgroup参数与系统一致，指定集群dns地址和域名

```bash
# 配置文件路径 /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
# 替换下面内容到10-kubeadm.conf文件中，注意修改cgroup参数与docker一致
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_SYSTEM_PODS_ARGS=--pod-manifest-path=/etc/kubernetes/manifests --allow-privileged=true"
Environment="KUBELET_NETWORK_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin"
Environment="KUBELET_DNS_ARGS=--cluster-dns=10.96.0.10 --cluster-domain=cluster.local"
Environment="KUBELET_AUTHZ_ARGS=--authorization-mode=Webhook --client-ca-file=/etc/kubernetes/pki/ca.crt"
# Value should match Docker daemon settings.
# Defaults are "cgroupfs" for Debian/Ubuntu/OpenSUSE and "systemd" for Fedora/CentOS/RHEL
Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=cgroupfs"
Environment="KUBELET_CADVISOR_ARGS=--cadvisor-port=0"
Environment="KUBELET_CERTIFICATE_ARGS=--rotate-certificates=true"
Environment="KUBE_PAUSE=--pod-infra-container-image=ccr.ccs.tencentyun.com/cherryleo/pause-amd64:3.0"
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_SYSTEM_PODS_ARGS $KUBELET_NETWORK_ARGS $KUBELET_DNS_ARGS $KUBELET_AUTHZ_ARGS $KUBELET_CGROUP_ARGS $KUBELET_CADVISOR_ARGS $KUBELET_CERTIFICATE_ARGS $KUBE_PAUSE $KUBELET_EXTRA_ARGS
```

重新载入kubelet 

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl stop kubelet
```

## Master节点安装

创建kubeadm启动配置文件，需要修改advertiseAddress为本机IP地址

```bash
# 创建master config.yaml文件，<ip>改为本机IP地址
$ cat >config.yaml <<EOF
apiVersion: kubeadm.k8s.io/v1alpha1
kind: MasterConfiguration
api:
    advertiseAddress: <ip>
networking:
    podSubnet: 10.244.0.0/16
apiServerCertSANs:
- <ip>
imageRepository: ccr.ccs.tencentyun.com/cherryleo
kubernetesVersion: v${KUBERNETES_VERSION}
EOF
```

kubeadm安装指令

```bash
$ sudo -E kubeadm init --config=config.yaml
```

安装成功后，配置kubectl

```bash
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

使用kubectl安装插件

```bash
# 网络插件安装，此处flannel网络
$ kubectl apply -f https://raw.githubusercontent.com/cherryleo/yamls/master/k8s-flannel/flannel.yaml

# dashboard插件安装
$ kubectl apply -f https://raw.githubusercontent.com/cherryleo/yamls/master/k8s-dashboard/kubernetes-dashboard.yaml

# 创建admin用户
$ kubectl apply -f https://raw.githubusercontent.com/cherryleo/yamls/master/k8s-dashboard/admin-user.yaml
```

查看集群状态

```bash
$ kubectl get nodes
NAME           STATUS    ROLES     AGE       VERSION
10-255-0-196   Ready     master    47m       v1.9.7

$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                   READY     STATUS    RESTARTS   AGE
kube-system   etcd-10-255-0-196                      1/1       Running   0          15m
kube-system   kube-apiserver-10-255-0-196            1/1       Running   0          15m
kube-system   kube-controller-manager-10-255-0-196   1/1       Running   0          15m
kube-system   kube-dns-7f5d7475f6-chfqz              3/3       Running   0          15m
kube-system   kube-flannel-ds-gjppn                  1/1       Running   0          10m
kube-system   kube-proxy-bbt6k                       1/1       Running   0          15m
kube-system   kube-scheduler-10-255-0-196            1/1       Running   0          15m
```

访问[https://master节点IP地址:30080](https://master节点IP地址:30080)进入登陆页面   

![dashboard-login.png](/kubernetes/dashboard-login.png)

获取登陆token
```bash
$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```

![dashboard-token.png](/kubernetes/dashboard-token.png)

![dashboard-index.png](/kubernetes/dashboard-index.png)

## Node节点安装

在Node节点重新执行依赖配置步骤，配置好系统后，在master节点获取加入集群的token信息，并通过token把node节点加入集群

```bash
# 在mster节点执行
$ sudo kubeadm token create --print-join-command
kubeadm join --token fddd11.35180a3132aa60b6 10.255.0.196:6443 --discovery-token-ca-cert-hash sha256:3c88d7639604c94304274bfe741e70039909c63da4c9db30229e987d7f443f34

# 在node节点执行
$ sudo kubeadm join --token fddd11.35180a3132aa60b6 10.255.0.196:6443 --discovery-token-ca-cert-hash sha256:3c88d7639604c94304274bfe741e70039909c63da4c9db30229e987d7f443f34
```

在master节点查看集群状态

```bash
$ kubectl get nodes
NAME           STATUS    ROLES     AGE       VERSION
10-255-0-196   Ready     master    47m       v1.9.7
10-255-0-252   Ready     <none>    2m        v1.9.7

$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                   READY     STATUS    RESTARTS   AGE
kube-system   etcd-10-255-0-196                      1/1       Running   0          47m
kube-system   kube-apiserver-10-255-0-196            1/1       Running   0          46m
kube-system   kube-controller-manager-10-255-0-196   1/1       Running   0          47m
kube-system   kube-dns-7f5d7475f6-chfqz              3/3       Running   0          47m
kube-system   kube-flannel-ds-gjppn                  1/1       Running   0          42m
kube-system   kube-flannel-ds-qbxzg                  1/1       Running   2          2m
kube-system   kube-proxy-bbt6k                       1/1       Running   0          47m
kube-system   kube-proxy-j9pks                       1/1       Running   0          2m
kube-system   kube-scheduler-10-255-0-196            1/1       Running   0          47m
```
