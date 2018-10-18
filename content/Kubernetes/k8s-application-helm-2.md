# K8S - 应用 - charts仓库

## 1. 私有charts仓库搭建

构建私有charts仓库很简单，只要有一个可以提供http/https的服务器即可，服务器需要提供index.yaml文件和打包好的charts文件

```bash
.
├── index.yaml
├── mysql-0.6.0.tgz
└── nginx-ingress-0.20.1.tgz
```

#### 1.1 创建chart并打包

##### 1.1.1 使用helm create创建一个chart

```bash
$ helm create myapp
Creating myapp
```
##### 1.1.2 查看chart文件

```bash
$ tree myapp/
myapp/
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   └── service.yaml
└── values.yaml
```

##### 1.1.3 使用helm package打包chart

```bash
$ helm package myapp/
Successfully packaged chart and saved it to: /tmp/myapp-0.1.0.tgz
```

#### 1.2 生成index.yaml文件

##### 1.2.1 把打包好的chart压缩包放到web服务器

```bash
.
├── index.yaml
├── myapp-0.1.0.tgz
├── mysql-0.6.0.tgz
└── nginx-ingress-0.20.1.tgz
```

##### 1.2.2 使用helm repo index重新生成index.yaml

```bash
$ helm repo index .
```

#### 1.3 小结

私有charts仓库的维护大致就是这几步，其中使用helm create创建完chart后，需要根据自己应用的实际情况修改chart目录下的文件，其中模版文件基于go模版语法



## 2. charts web ui

monocular项目提供了charts仓库的web ui，https://github.com/kubernetes-helm/monocular

monocular使用helm进行安装，依赖mongodb，并且需要保证nginx-ingress已在k8s集群中安装

#### 2.1 添加chart仓库

```bash
$ helm repo add cherryleo https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/charts
```

#### 2.2 安装ingress

```bash
$ helm install cherryleo/nginx-ingress --set controller.image.repository=ccr.ccs.tencentyun.com/quay.io/nginx-ingress-controller --set controller.image.tag=0.15.0 --set defaultBackend.image.repository=ccr.ccs.tencentyun.com/k8s.io/defaultbackend --set defaultBackend.image.tag=1.4
```

#### 2.3 安装monocular

```bash
$ helm install cherryleo/monocular --name monocular
```

#### 2.4 查看monocular

```bash
$ kubectl get pods
NAME                                                              READY     STATUS    RESTARTS   AGE
giggly-grasshopper-nginx-ingress-controller-8965d964f-58zwr       1/1       Running   0          2h
giggly-grasshopper-nginx-ingress-default-backend-58954bc7chdvxz   1/1       Running   0          2h
monocular-mongodb-6c9b897fdf-nh49n                                1/1       Running   0          4m
monocular-monocular-api-5697776746-h4j9z                          1/1       Running   2          4m
monocular-monocular-prerender-6866b8c48d-8zdqt                    1/1       Running   0          4m
monocular-monocular-ui-75f899bf44-x96hr                           1/1       Running   0          4m
```

#### 2.5 查看ingress端口

monocular必须通过ingress进行访问

```bash
$ kubectl get svc | grep ingress
my-ingress-controller        LoadBalancer   10.100.66.211    <pending>     80:31511/TCP,443:32308/TCP   3h
```

#### 2.6 访问monocular

![](https://fileserver-1253732882.cos.ap-chongqing.myqcloud.com/pic/k8s-application-monocular-1.png)

![](https://fileserver-1253732882.cos.ap-chongqing.myqcloud.com/pic/k8s-application-monocular-2.png)

![](https://fileserver-1253732882.cos.ap-chongqing.myqcloud.com/pic/k8s-application-monocular-3.png)