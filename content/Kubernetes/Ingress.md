---
weight: 2100
title: "Ingress"
date: "2018-11-12"
lastmod: "2018-11-12"
description:  "Ingress安装，使用Ingress暴露Kubernetes服务"
categories:  ["kubernetes"]
tags: ["kubernetes"]
---

## 简介

使用nginx-ingress-controller暴露k8s内部服务，需要进行dns绑定，本地测试可以修改hosts进行测试

nginx-ingress-controller官方镜像同步于腾讯云，无需翻墙，加速访问


## 安装

```shell
$ kubectl apply -f https://raw.githubusercontent.com/cherryleo/k8s-apps/master/ingress-nginx/mandatory.yaml
$ kubectl apply -f https://raw.githubusercontent.com/cherryleo/k8s-apps/master/ingress-nginx/service-nodeport.yaml
```

## 验证

查看nginx-ingress-controller

```shell
# kubectl get pod -n ingress-nginx
NAME                                       READY     STATUS    RESTARTS   AGE
default-http-backend-85db865f85-x6782      1/1       Running   0          8m
nginx-ingress-controller-67cc957bc-s9t7q   1/1       Running   0          8m
```

浏览器访问nginx-ingress-controller

nginx-ingress-controller使用nodeport暴露自身，端口30090，外部流量通过nginx-ingress转发到内部服务

![](https://fileserver-1253732882.cos.ap-chongqing.myqcloud.com/pic/k8s-ingress-self-service.png)

创建dashboard的ingress描述文件

需要绑定域名dashboard.myk8s.org到k8s集群地址，ingress会把访问dashboard.myk8s.org的流量转发到dashboard服务

```yaml
# ingress-dashboard.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/proxy-read-timeout: "600"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "600"
    nginx.ingress.kubernetes.io/secure-backends: "true"
    nginx.org/ssl-backends: "kubernetes-dashboard"
    ingress.kubernetes.io/ssl-passthrough: "true"
  name: dashboard
  namespace: kube-system
spec:
  tls:
  - hosts:
    - dashboard.myk8s.org
    secretName: kubernetes-dashboard-certs
  rules:
  - host: dashboard.myk8s.org
    http:
      paths:
      - backend:
          serviceName: kubernetes-dashboard
          servicePort: 443
        path: /
```

创建dashboard ingress

```shell
kubectl create -f ingress-dashboard.yaml
```

基于域名访问，需要绑定dns或者修改hosts进行测试

![](https://fileserver-1253732882.cos.ap-chongqing.myqcloud.com/pic/k8s-ingress-dashboard.png)