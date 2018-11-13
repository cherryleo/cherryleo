---
weight: 4000
title: "Helm（一）"
date: "2018-11-12"
lastmod: "2018-11-12"
description:  "使用Helm管理应用，安装Helm，Helm简单使用"
categories:  ["kubernetes"]
tags: ["kubernetes"]
---

## 简介

Helm源自Deis公司（微软收购）开发并托管于Github上的开源项目Helm Classic。目前隶属于CNCF，是Kubernetes官方认可的应用管理工具

Helm 采用客户端/服务器架构，由客户端工具helm，服务端组建tiller，charts仓库组成。

## 安装

helm客户端安装

直接从官方仓库下载 https://github.com/kubernetes/helm/releases 如果官方下载速度比较慢，可以从下面下载地址下载，从官方同步了一份存储于腾讯云对象存储

```shell
$ wget https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/helm-v2.9.1-linux-amd64.tar.gz
```

解压并复制helm文件到可执行路径下

```shell
$ tar -zxvf helm-v2.9.1-linux-amd64.tar.gz
$ sudo cp linux-amd64/helm /usr/local/bin
```

服务端tiller安装

安装tiller时，会去google镜像仓库拉取镜像，由于访问google速度较慢，同步了一份镜像存储于腾讯云镜像仓库，安装过程需要在可以访问
kubernetes集群的主机上进行，执行下面指令进行安装

```shell
$ helm init --tiller-image ccr.ccs.tencentyun.com/cherryleo/tiller:v2.9.1 --skip-refresh
```

查看tiller是否在集群上成功运行，只要tiller处于运行状态，则表示安装成功

```shell
$ kubectl get pods -n kube-system | grep tiller
tiller-deploy-7594bf7b76-qfzss           1/1       Running   0          1d
```

开启rbac授权

```shell
$ kubectl create serviceaccount --namespace kube-system tiller
$ kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
$ kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
```

## Charts简介

如果我们需要在k8s上面跑应用，需要事先写好相关资源的yaml文件，比如deployment.yaml，service.yaml等，chart就是一个应用需要的资源文件集合，简单来说，chart就是把k8s应用所需的deployment，service，pvc，secret等组合在一起并作为模版使用

helm使用chart管理应用，通过chart可以方便的安装一个应用需要所有的资源，也很容易安装应用需要的依赖

```shell
# mysql chart
.
├── charts
├── Chart.yaml
├── README.md
├── templates
│   ├── configurationFiles-configmap.yaml
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── initializationFiles-configmap.yaml
│   ├── NOTES.txt
│   ├── pvc.yaml
│   ├── secrets.yaml
│   └── svc.yaml
└── values.yaml
```

charts仓库即应用仓库，官方应用仓库托管于google。可以通过helm指令增加第三方仓库或私有charts仓库

查看charts仓库

```shell
$ helm repo list
NAME  	URL                                             
stable	https://kubernetes-charts.storage.googleapis.com
local 	http://127.0.0.1:8879/charts 
```

添加charts仓库

```shell
$ helm repo add cherryleo https://fileserver-1253732882.cos.ap-chongqing.myqcloud.com/charts
$ helm repo list
NAME     	URL                                                               
stable   	https://kubernetes-charts.storage.googleapis.com                  
local    	http://127.0.0.1:8879/charts                                      
cherryleo	https://fileserver-1253732882.cos.ap-chongqing.myqcloud.com/charts
```

使用helm安装一个nginx-ingress应用，安装指令加了一些镜像仓库地址的参数

```shell
$ helm install cherryleo/nginx-ingress --set controller.image.repository=ccr.ccs.tencentyun.com/quay.io/nginx-ingress-controller --set controller.image.tag=0.15.0 --set defaultBackend.image.repository=ccr.ccs.tencentyun.com/k8s.io/defaultbackend --set defaultBackend.image.tag=1.4
```

查看应用helm安装的应用

```shell
$ helm list
NAME              	REVISION	UPDATED                 	STATUS  	CHART               	NAMESPACE
giggly-grasshopper	1       	Wed Jun  6 15:06:24 2018	DEPLOYED	nginx-ingress-0.20.1	default  
```

更多参考资料

helm官方仓库：https://github.com/kubernetes/helm

chart官方仓库：https://github.com/kubernetes/charts

helm官方文档：https://docs.helm.sh