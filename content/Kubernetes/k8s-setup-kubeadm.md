---
title: "K8S - 安装 - kubeadm"
date: 2018-10-09T11:39:43+08:00
weight: 100
---

# K8S - 安装 - kubeadm

## 1. 简介

kubeadm是k8s官方工具，用来快速安装k8s集群。整套工具由kubeadm，kubelet，kubernetes-cni和kubectl包组成，除kubelet以二进制方式运行外，其余组件均作为k8s静态pod启动（镜像均托管谷歌与gcr.io仓库）。

目前kubeadm支持通过配置文件imageRepository指定镜像仓库地址，个人在使用和学习过程中同步一些官方镜像于腾讯云镜像仓库，并写了一些自动部署的脚本，以便快速部署k8s集群。

本文档详细介绍使用kubadm部署k8s集群完整步骤



## 2.依赖

#### 2.1 操作系统

支持Ubuntu 16.04，CentOS 7+，amd64，master节点配置2核2G以上，安装以下软件包

```console
# CentOS7
$ sudo yum install ebtables ethtool iproute iptables socat util-linux wget -y

# Ubuntu16.04
$ sudo apt-get install ebtables ethtool iproute iptables socat util-linux wget -y
```

#### 2.2 安装docker，docker版本小于等于17

```console
# CentOS7安装docker-ce-17.03
$ sudo wget -O - https://raw.githubusercontent.com/cherryleo/scripts/master/centos7-install-docker.sh | sudo sh

# Ubuntu16.04安装docker-ce-17.03
$ sudo wget -O - https://raw.githubusercontent.com/cherryleo/scripts/master/ubuntu16.04-install-docker.sh | sudo sh
```

#### 2.3 系统设置

```console
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

#### 2.4 安装kubeadm

```console
$ wget -O - https://raw.githubusercontent.com/cherryleo/cherryleo/master/install-k8s-packages.sh | sudo -E bash
```

#### 2.5 配置kubeadm

##### 2.5.1 查看docker cgroup driver

```console
$ sudo docker info | grep -i cgroup
```

##### 2.5.2 修改kubeadm配置文件 

```shell
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
##### 2.5.3 重新载入kubelet 

```console
$ sudo systemctl daemon-reload
$ sudo systemctl stop kubelet
```



## 3. kubeadm安装k8s集群

#### 3.1 安装k8s master节点

##### 3.1.1 配置文件

```console
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

##### 3.1.2 执行安装

```console
$ sudo -E kubeadm init --config=config.yaml
```

##### 3.1.3 # 安装成功后，创建kubectl配置文件

```console
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

##### 3.1.4 安装插件

```console
# 网络插件安装，此处flannel网络
$ kubectl apply -f https://raw.githubusercontent.com/cherryleo/k8s-apps/master/k8s-flannel/flannel.yaml
# dashboard安装
$ kubectl apply -f https://raw.githubusercontent.com/cherryleo/k8s-apps/master/k8s-dashboard/kubernetes-dashboard.yaml
# 创建admin用户
$ kubectl apply -f https://raw.githubusercontent.com/cherryleo/k8s-apps/master/k8s-dashboard/admin-user.yaml
```

##### 3.1.5 查看集群状态

```console
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

##### 3.1.6 访问[https://ip:30080](https://ip:30080)进入登陆页面

![](https://fileserver-1253732882.cos.ap-chongqing.myqcloud.com/pic/k8s-dashboard-login.png)

```console
# 获取token
$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```

![](https://fileserver-1253732882.cos.ap-chongqing.myqcloud.com/pic/k8s-dashboard-token.png)

![](https://fileserver-1253732882.cos.ap-chongqing.myqcloud.com/pic/k8s-dashboard.png)

#### 3.2 node节点安装

##### 3.2.1 执行第二大步骤，进行node节点初始化

##### 3.2.2 获取加入集群token信息

```console
# 在mster节点执行
$ sudo kubeadm token create --print-join-command
kubeadm join --token fddd11.35180a3132aa60b6 10.255.0.196:6443 --discovery-token-ca-cert-hash sha256:3c88d7639604c94304274bfe741e70039909c63da4c9db30229e987d7f443f34
```

##### 3.2.3 把node节点加入集群

```console
# 在node节点执行
$ sudo kubeadm join --token fddd11.35180a3132aa60b6 10.255.0.196:6443 --discovery-token-ca-cert-hash sha256:3c88d7639604c94304274bfe741e70039909c63da4c9db30229e987d7f443f34
```

##### 3.2.4 在master节点查看集群状态

```console
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
