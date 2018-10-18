# k8s官方资源同步列表

## 1. 官方镜像

#### 1.1 etcd

```console
ccr.ccs.tencentyun.com/cherryleo/etcd-amd64:3.1.10
ccr.ccs.tencentyun.com/cherryleo/etcd-amd64:3.1.11
ccr.ccs.tencentyun.com/cherryleo/etcd-amd64:3.1.12
ccr.ccs.tencentyun.com/cherryleo/etcd-amd64:3.1.13
ccr.ccs.tencentyun.com/cherryleo/etcd-amd64:3.1.14
ccr.ccs.tencentyun.com/cherryleo/etcd-amd64:3.1.15
ccr.ccs.tencentyun.com/cherryleo/etcd-amd64:3.1.16
ccr.ccs.tencentyun.com/cherryleo/etcd-amd64:3.1.17
```

#### 1.2 kube

```console
# pause
ccr.ccs.tencentyun.com/cherryleo/pause-amd64:3.0

# apiserver
ccr.ccs.tencentyun.com/cherryleo/kube-apiserver-amd64:v1.9.0
ccr.ccs.tencentyun.com/cherryleo/kube-apiserver-amd64:v1.9.1
ccr.ccs.tencentyun.com/cherryleo/kube-apiserver-amd64:v1.9.2
ccr.ccs.tencentyun.com/cherryleo/kube-apiserver-amd64:v1.9.3
ccr.ccs.tencentyun.com/cherryleo/kube-apiserver-amd64:v1.9.4
ccr.ccs.tencentyun.com/cherryleo/kube-apiserver-amd64:v1.9.5
ccr.ccs.tencentyun.com/cherryleo/kube-apiserver-amd64:v1.9.6
ccr.ccs.tencentyun.com/cherryleo/kube-apiserver-amd64:v1.9.7
ccr.ccs.tencentyun.com/cherryleo/kube-apiserver-amd64:v1.9.8
ccr.ccs.tencentyun.com/cherryleo/kube-apiserver-amd64:v1.9.9
ccr.ccs.tencentyun.com/cherryleo/kube-apiserver-amd64:v1.10.0
ccr.ccs.tencentyun.com/cherryleo/kube-apiserver-amd64:v1.10.1
ccr.ccs.tencentyun.com/cherryleo/kube-apiserver-amd64:v1.10.2
ccr.ccs.tencentyun.com/cherryleo/kube-apiserver-amd64:v1.10.3
ccr.ccs.tencentyun.com/cherryleo/kube-apiserver-amd64:v1.10.4
ccr.ccs.tencentyun.com/cherryleo/kube-apiserver-amd64:v1.10.5
ccr.ccs.tencentyun.com/cherryleo/kube-apiserver-amd64:v1.11.0
ccr.ccs.tencentyun.com/cherryleo/kube-apiserver-amd64:v1.11.1

# controller-manager
ccr.ccs.tencentyun.com/cherryleo/kube-controller-manager-amd64:v1.9.0
ccr.ccs.tencentyun.com/cherryleo/kube-controller-manager-amd64:v1.9.1
ccr.ccs.tencentyun.com/cherryleo/kube-controller-manager-amd64:v1.9.2
ccr.ccs.tencentyun.com/cherryleo/kube-controller-manager-amd64:v1.9.3
ccr.ccs.tencentyun.com/cherryleo/kube-controller-manager-amd64:v1.9.4
ccr.ccs.tencentyun.com/cherryleo/kube-controller-manager-amd64:v1.9.5
ccr.ccs.tencentyun.com/cherryleo/kube-controller-manager-amd64:v1.9.6
ccr.ccs.tencentyun.com/cherryleo/kube-controller-manager-amd64:v1.9.7
ccr.ccs.tencentyun.com/cherryleo/kube-controller-manager-amd64:v1.9.8
ccr.ccs.tencentyun.com/cherryleo/kube-controller-manager-amd64:v1.9.9
ccr.ccs.tencentyun.com/cherryleo/kube-controller-manager-amd64:v1.10.0
ccr.ccs.tencentyun.com/cherryleo/kube-controller-manager-amd64:v1.10.1
ccr.ccs.tencentyun.com/cherryleo/kube-controller-manager-amd64:v1.10.2
ccr.ccs.tencentyun.com/cherryleo/kube-controller-manager-amd64:v1.10.3
ccr.ccs.tencentyun.com/cherryleo/kube-controller-manager-amd64:v1.10.4
ccr.ccs.tencentyun.com/cherryleo/kube-controller-manager-amd64:v1.10.5
ccr.ccs.tencentyun.com/cherryleo/kube-controller-manager-amd64:v1.11.0
ccr.ccs.tencentyun.com/cherryleo/kube-controller-manager-amd64:v1.11.1

# scheduler
ccr.ccs.tencentyun.com/cherryleo/kube-scheduler-amd64:v1.9.0
ccr.ccs.tencentyun.com/cherryleo/kube-scheduler-amd64:v1.9.1
ccr.ccs.tencentyun.com/cherryleo/kube-scheduler-amd64:v1.9.2
ccr.ccs.tencentyun.com/cherryleo/kube-scheduler-amd64:v1.9.3
ccr.ccs.tencentyun.com/cherryleo/kube-scheduler-amd64:v1.9.4
ccr.ccs.tencentyun.com/cherryleo/kube-scheduler-amd64:v1.9.5
ccr.ccs.tencentyun.com/cherryleo/kube-scheduler-amd64:v1.9.6
ccr.ccs.tencentyun.com/cherryleo/kube-scheduler-amd64:v1.9.7
ccr.ccs.tencentyun.com/cherryleo/kube-scheduler-amd64:v1.9.8
ccr.ccs.tencentyun.com/cherryleo/kube-scheduler-amd64:v1.9.9
ccr.ccs.tencentyun.com/cherryleo/kube-scheduler-amd64:v1.10.0
ccr.ccs.tencentyun.com/cherryleo/kube-scheduler-amd64:v1.10.1
ccr.ccs.tencentyun.com/cherryleo/kube-scheduler-amd64:v1.10.2
ccr.ccs.tencentyun.com/cherryleo/kube-scheduler-amd64:v1.10.3
ccr.ccs.tencentyun.com/cherryleo/kube-scheduler-amd64:v1.10.4
ccr.ccs.tencentyun.com/cherryleo/kube-scheduler-amd64:v1.10.5
ccr.ccs.tencentyun.com/cherryleo/kube-scheduler-amd64:v1.11.0
ccr.ccs.tencentyun.com/cherryleo/kube-scheduler-amd64:v1.11.1

# proxy
ccr.ccs.tencentyun.com/cherryleo/kube-proxy-amd64:v1.9.0
ccr.ccs.tencentyun.com/cherryleo/kube-proxy-amd64:v1.9.1
ccr.ccs.tencentyun.com/cherryleo/kube-proxy-amd64:v1.9.2
ccr.ccs.tencentyun.com/cherryleo/kube-proxy-amd64:v1.9.3
ccr.ccs.tencentyun.com/cherryleo/kube-proxy-amd64:v1.9.4
ccr.ccs.tencentyun.com/cherryleo/kube-proxy-amd64:v1.9.5
ccr.ccs.tencentyun.com/cherryleo/kube-proxy-amd64:v1.9.6
ccr.ccs.tencentyun.com/cherryleo/kube-proxy-amd64:v1.9.7
ccr.ccs.tencentyun.com/cherryleo/kube-proxy-amd64:v1.9.8
ccr.ccs.tencentyun.com/cherryleo/kube-proxy-amd64:v1.9.9
ccr.ccs.tencentyun.com/cherryleo/kube-proxy-amd64:v1.10.0
ccr.ccs.tencentyun.com/cherryleo/kube-proxy-amd64:v1.10.1
ccr.ccs.tencentyun.com/cherryleo/kube-proxy-amd64:v1.10.2
ccr.ccs.tencentyun.com/cherryleo/kube-proxy-amd64:v1.10.3
ccr.ccs.tencentyun.com/cherryleo/kube-proxy-amd64:v1.10.4
ccr.ccs.tencentyun.com/cherryleo/kube-proxy-amd64:v1.10.5
ccr.ccs.tencentyun.com/cherryleo/kube-proxy-amd64:v1.11.0
ccr.ccs.tencentyun.com/cherryleo/kube-proxy-amd64:v1.11.1
```

#### 1.3 kube-dns

```console
# k8s-dns-kube-dns
ccr.ccs.tencentyun.com/cherryleo/k8s-dns-kube-dns-amd64:1.14.6
ccr.ccs.tencentyun.com/cherryleo/k8s-dns-kube-dns-amd64:1.14.7
ccr.ccs.tencentyun.com/cherryleo/k8s-dns-kube-dns-amd64:1.14.8
ccr.ccs.tencentyun.com/cherryleo/k8s-dns-kube-dns-amd64:1.14.9
ccr.ccs.tencentyun.com/cherryleo/k8s-dns-kube-dns-amd64:1.14.10

# k8s-dns-sidecar
ccr.ccs.tencentyun.com/cherryleo/k8s-dns-sidecar-amd64:1.14.6
ccr.ccs.tencentyun.com/cherryleo/k8s-dns-sidecar-amd64:1.14.7
ccr.ccs.tencentyun.com/cherryleo/k8s-dns-sidecar-amd64:1.14.8
ccr.ccs.tencentyun.com/cherryleo/k8s-dns-sidecar-amd64:1.14.9
ccr.ccs.tencentyun.com/cherryleo/k8s-dns-sidecar-amd64:1.14.10

# k8s-dns-dnsmasq-nanny-amd64
ccr.ccs.tencentyun.com/cherryleo/k8s-dns-dnsmasq-nanny-amd64:1.14.6
ccr.ccs.tencentyun.com/cherryleo/k8s-dns-dnsmasq-nanny-amd64:1.14.7
ccr.ccs.tencentyun.com/cherryleo/k8s-dns-dnsmasq-nanny-amd64:1.14.8
ccr.ccs.tencentyun.com/cherryleo/k8s-dns-dnsmasq-nanny-amd64:1.14.9
ccr.ccs.tencentyun.com/cherryleo/k8s-dns-dnsmasq-nanny-amd64:1.14.10
```


#### 1.4 dashboard

```console
ccr.ccs.tencentyun.com/cherryleo/kubernetes-dashboard-amd64:v1.8.2
ccr.ccs.tencentyun.com/cherryleo/kubernetes-dashboard-amd64:v1.8.3
```



## 2. rpm包(centos7+)

```console
# kubeadm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm-1.9.0-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm-1.9.1-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm-1.9.2-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm-1.9.3-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm-1.9.4-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm-1.9.5-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm-1.9.6-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm-1.9.7-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm-1.9.8-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm-1.9.9-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm-1.10.0-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm-1.10.1-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm-1.10.2-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm-1.10.3-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm-1.10.4-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm-1.10.5-0.x86_64.rpm

# kubelet
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet-1.9.0-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet-1.9.1-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet-1.9.2-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet-1.9.3-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet-1.9.4-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet-1.9.5-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet-1.9.6-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet-1.9.7-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet-1.9.8-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet-1.9.9-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet-1.10.0-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet-1.10.1-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet-1.10.2-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet-1.10.3-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet-1.10.4-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet-1.10.5-0.x86_64.rpm

# kubectl
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl-1.9.0-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl-1.9.1-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl-1.9.2-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl-1.9.3-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl-1.9.4-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl-1.9.5-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl-1.9.6-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl-1.9.7-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl-1.9.8-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl-1.9.9-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl-1.10.0-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl-1.10.1-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl-1.10.2-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl-1.10.3-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl-1.10.4-0.x86_64.rpm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl-1.10.5-0.x86_64.rpm

# kubernetes-cni
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubernetes-cni-0.6.0-0.x86_64.rpm
```



## 3. deb包(ubuntu16.04)

```console
# kubeadm
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm_1.9.0-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm_1.9.1-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm_1.9.2-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm_1.9.3-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm_1.9.4-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm_1.9.5-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm_1.9.6-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm_1.9.7-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm_1.9.8-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm_1.9.9-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm_1.10.0-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm_1.10.1-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm_1.10.2-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm_1.10.3-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm_1.10.4-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubeadm_1.10.5-00_amd64.deb

# kubelet
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet_1.9.0-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet_1.9.1-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet_1.9.2-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet_1.9.3-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet_1.9.4-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet_1.9.5-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet_1.9.6-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet_1.9.7-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet_1.9.8-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet_1.9.9-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet_1.10.0-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet_1.10.1-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet_1.10.2-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet_1.10.3-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet_1.10.4-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubelet_1.10.5-00_amd64.deb

# kubectl
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl_1.9.0-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl_1.9.1-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl_1.9.2-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl_1.9.3-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl_1.9.4-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl_1.9.5-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl_1.9.6-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl_1.9.7-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl_1.9.8-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl_1.9.9-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl_1.10.0-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl_1.10.1-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl_1.10.2-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl_1.10.3-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl_1.10.4-00_amd64.deb
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubectl_1.10.5-00_amd64.deb

# kubernetes-cni
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubernetes-cni_0.6.0-00_amd64.deb
```



## 4. kubernetes官方编译包

```console
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubernetes-server-linux-amd64-v1.10.0.tar.gz
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubernetes-server-linux-amd64-v1.10.1.tar.gz
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubernetes-server-linux-amd64-v1.10.2.tar.gz
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubernetes-server-linux-amd64-v1.10.3.tar.gz
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubernetes-server-linux-amd64-v1.10.4.tar.gz
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubernetes-server-linux-amd64-v1.10.5.tar.gz
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubernetes-server-linux-amd64-v1.11.0.tar.gz
https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/kubernetes-server-linux-amd64-v1.11.1.tar.gz
```

