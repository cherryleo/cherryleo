# k8s使用ingress暴露服务

## 1. 简介

使用nginx-ingress-controller暴露k8s内部服务，需要进行dns绑定，本地测试可以修改hosts进行测试

nginx-ingress-controller官方镜像同步于腾讯云，无需翻墙，加速访问



## 2. 安装

```console
kubectl apply -f https://raw.githubusercontent.com/cherryleo/k8s-apps/master/ingress-nginx/mandatory.yaml
kubectl apply -f https://raw.githubusercontent.com/cherryleo/k8s-apps/master/ingress-nginx/service-nodeport.yaml
```



## 3. 验证

#### 3.1 查看nginx-ingress-controller

```console
# kubectl get pod -n ingress-nginx
NAME                                       READY     STATUS    RESTARTS   AGE
default-http-backend-85db865f85-x6782      1/1       Running   0          8m
nginx-ingress-controller-67cc957bc-s9t7q   1/1       Running   0          8m
```

#### 3.2 浏览器访问nginx-ingress-controller

nginx-ingress-controller使用nodeport暴露自身，端口30090，外部流量通过nginx-ingress转发到内部服务

![](https://fileserver-1253732882.cos.ap-chongqing.myqcloud.com/pic/k8s-ingress-self-service.png)

#### 3.3 创建dashboard的ingress描述文件

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

#### 3.4 创建dashboard ingress

```console
kubectl create -f ingress-dashboard.yaml
```

#### 3.5 基于域名访问

需要绑定dns或者修改hosts进行测试

![](https://fileserver-1253732882.cos.ap-chongqing.myqcloud.com/pic/k8s-ingress-dashboard.png)