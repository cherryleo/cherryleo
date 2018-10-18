# K8S - 安装 - kubeadm高可用

## 1. 简介

使用kubeadm部署高可用k8s集群，所有资源均托管于腾讯云，无需翻墙

master主机3台，worker主机一台

高可用的核心在于kube-apiserver的高可用，也就是需要有负载均衡器对多个kube-apiserver进行代理，同时保证kubelet和kube-proxy组件指向的是负载均衡地址而不是某个固定的kube-apiserver地址

由于使用kubeadm安装k8s时，集群使用的tls加密认证，所以在生成证书处需要指定的是负载均衡的IP地址

另外，etcd需要创建集群，kube-scheduler和kube-controller-manager集群是自选举



## 2. 拓扑图

![](https://fileserver-1253732882.cos.ap-chongqing.myqcloud.com/pic/k8s-ha.png)



## 3. 部署etcd集群

#### 3.1 简介

部署etcd-v3.1.11集群，基于http的etcd集群，未启用证书认证和通信加密

etcd集群部署在3台master节点上，使用系统服务启动

#### 3.2 安装etcd 

##### 3.2.1 三台master节点均执行下面指令下载二进制文件并安装

```console
wget https://cherryleo-1253732882.cos.ap-chengdu.myqcloud.com/etcd-v3.1.11-linux-amd64.tar.gz && tar -zxf etcd-v3.1.11-linux-amd64.tar.gz
cp etcd-v3.1.11-linux-amd64/etcd* /usr/local/bin
```

##### 3.2.2 设置环境变量，前三个变量为主机地址，第四个变量为当前主机ID

master0节点设置

```console
export etcd0=10.250.0.6
export etcd1=10.250.0.7
export etcd2=10.250.0.8
export current_host=etcd0
```

master1节点设置

```console
export etcd0=10.250.0.6
export etcd1=10.250.0.7
export etcd2=10.250.0.8
export current_host=etcd1
```

master2节点设置

```console
export etcd0=10.250.0.6
export etcd1=10.250.0.7
export etcd2=10.250.0.8
export current_host=etcd2
```

##### 3.2.3 创建service，三台master节点直接复制下面内容执行

```console
# cat >/etc/systemd/system/etcd.service <<EOF
[Unit]
Description=etcd
Documentation=https://github.com/coreos/etcd
Conflicts=etcd.service
Conflicts=etcd2.service

[Service]
Type=notify
Restart=always
RestartSec=5s
LimitNOFILE=40000
TimeoutStartSec=0

ExecStart=/usr/local/bin/etcd --name ${current_host} --data-dir /var/lib/etcd --listen-client-urls http://${!current_host}:2379,http://127.0.0.1:2379 --advertise-client-urls http://${!current_host}:2379 --listen-peer-urls http://${!current_host}:2380 --initial-advertise-peer-urls http://${!current_host}:2380 --initial-cluster etcd0=http://${etcd0}:2380,etcd1=http://${etcd1}:2380,etcd2=http://${etcd2}:2380 --initial-cluster-token my-etcd-token --initial-cluster-state new

[Install]
WantedBy=multi-user.target
EOF
```

##### 3.2.4 激活并启动service

三台master节点均需要执行，最后一步骤，先启动的节点会等待，直到最后一台etcd启动，集群才初始化成功

```console
# systemctl daemon-reload
# systemctl enable etcd.service
# systemctl start etcd
```

##### 3.2.5 集群健康检查

```console
# etcdctl cluster-health
member 3304230ad5dab676 is healthy: got healthy result from http://10.250.0.7:2379
member 3d72a28f2042bf01 is healthy: got healthy result from http://10.250.0.6:2379
member 958be7174df0bec6 is healthy: got healthy result from http://10.250.0.8:2379
cluster is healthy
# etcdctl member list
```



## 4 部署keepalived

#### 4.1 简介

使用keepalived作高可用，keepalived的虚拟ip会指向三台master节点的ip，但仅保证高可用，并不进行负载均衡，如果需要负载均衡可以配合lvs，nginx等一起使用

#### 4.2 安装keepalived

##### 4.2.1 直接使用yum或apt进行安装，三台master节点均要安装

```console
# yum install keepalived -y
```

##### 4.2.2 配置环境变量，三台master节点均要配置，需要修改role，interface，vip变量

interface修改为本机通信网口名称，vip为keepalived虚拟ip地址，需要保证虚拟ip未被占用

master0节点设置，master0作为keepalived的MASTER节点，网口eth0，keepalived虚拟ip为10.255.0.10

```console
# export role=MASTER
# export interface=eth0
# export priority=101
# export vip=10.250.0.10
```

master1，master2节点设置，master1，master2节点作为keepalived的BACKUP节点，设置一样

```console
# export role=BACKUP
# export interface=eth0
# export priority=100
# export vip=10.250.0.10
```

##### 4.2.3 创建keepalived配置文件，三台master节点直接复制下面内容执行

```Console
# cat >/etc/keepalived/keepalived.conf << EOF
! Configuration File for keepalived
global_defs {
  router_id LVS_DEVEL
}

vrrp_script check_apiserver {
  script "/etc/keepalived/check_apiserver.sh"
  interval 3
  weight -2
  fall 10
  rise 2
}

vrrp_instance VI_1 {
    state ${role}
    interface ${interface}
    virtual_router_id 51
    priority ${priority}
    authentication {
        auth_type PASS
        auth_pass 4be37dc3b4c90194d1600c483e10ad1d
    }
    virtual_ipaddress {
        ${vip}
    }
    track_script {
        check_apiserver
    }
}
EOF
```

##### 4.2.4 apiserver检查配置，三台master节点直接复制下面内容执行

```shell
cat >/etc/keepalived/check_apiserver.sh << EOF
#!/bin/sh

errorExit() {
    echo "*** $*" 1>&2
    exit 1
}

curl --silent --max-time 2 --insecure https://localhost:6443/ -o /dev/null || errorExit "Error GET https://localhost:6443/"
if ip addr | grep -q ${vip}; then
    curl --silent --max-time 2 --insecure https://${vip}:6443/ -o /dev/null || errorExit "Error GET https://${vip}:6443/"
fi
EOF
```

##### 4.2.5 重启keepalived，三台master节点均需要重启

```console
# systemctl restart keepalived
```

##### 4.2.6 验证keepalived，查看eth0网口是否多出虚拟ip地址，三台master节点均需验证

```console
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:50:56:64:85:4a brd ff:ff:ff:ff:ff:ff
    inet 10.250.0.6/28 brd 10.250.0.15 scope global eth0
       valid_lft forever preferred_lft forever
    inet 10.250.0.10/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::250:56ff:fe64:854a/64 scope link 
       valid_lft forever preferred_lft forever
```



## 5. 部署k8s集群

#### 5.1 安装必要系统组件，支持ubuntu16.04/CentOS 7+，amd64，master节点配置2核2G以上

```console
# CentOS
yum install ebtables ethtool iproute iptables socat util-linux wget -y

# Ubuntu
apt-get install ebtables ethtool iproute iptables socat util-linux wget -y
```

#### 5.2 安装docker，已安装略过此步骤

```console
# CentOS7安装docker-ce-17.03
wget -O - https://raw.githubusercontent.com/cherryleo/cherryleo/master/centos7-install-docker.sh | sudo sh

# Ubuntu16.04安装docker-ce-17.03
wget -O - https://raw.githubusercontent.com/cherryleo/cherryleo/master/ubuntu16.04-install-docker.sh | sudo sh
```

#### 5.3 系统设置

```shell
# 关闭swap
swapoff -a

# 关闭防火墙，如果不关防火墙，确保8080，6443，10250端口开放
systemctl stop firewalld
systemctl disable firewalld

# 修改网络参数
sysctl net.bridge.bridge-nf-call-iptables=1

# 设置环境变量，k8s安装版本，1.9.0 - 1.9.9, 1.10.0 - 1.10.5
export KUBERNETES_VERSION="1.9.7"
```

#### 5.4 安装kubeadm

```console
wget -O - https://raw.githubusercontent.com/cherryleo/cherryleo/master/install-k8s-packages.sh | sudo -E bash
```

#### 5.5 配置kubeadm

##### 5.5.1 查看docker cgroup dirver

```console
docker info | grep -i cgroup
```

##### 5.5.2 修改kubeadm配置文件

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

##### 5.5.3 重新载入kebelet

```console
systemctl daemon-reload
systemctl stop kubelet
```

#### 5.6 安装k8s master节点，确保三台master节点均完整执行5.1 - 5.5步骤

##### 5.6.1 创建安装配置文件，三台master节点直接复制下面内容执行

```Console
cat >config.yaml <<EOF
apiVersion: kubeadm.k8s.io/v1alpha1
kind: MasterConfiguration
api:
    advertiseAddress: ${!current_host}
etcd:
    endpoints:
    - http://${etcd0}:2379
    - http://${etcd1}:2379
    - http://${etcd2}:2379
networking:
    podSubnet: 10.244.0.0/16
apiServerCertSANs:
- ${vip}
apiServerExtraArgs:
    apiserver-count: "3"
imageRepository: ccr.ccs.tencentyun.com/cherryleo
kubernetesVersion: v${KUBERNETES_VERSION}
EOF
```

##### 5.6.2 使用kubeadm先安装master0节点，先安装master0节点，其余两个节点不进行安装

```console
kubeadm init --config=config.yaml 
```

##### 5.6.3 把master0节点生成的证书文件拷贝到master1和master2节点

```console
mkdir -vp /etc/kubernetes/pki
scp root@10.250.0.6:/etc/kubernetes/pki/* /etc/kubernetes/pki/
rm /etc/kubernetes/pki/apiserver.*
```

##### 5.6.4 使用kubeadm安装master1，master2节点，两个节点可以同步进行安装

```console
kubeadm init --config=config.yaml 
```

##### 5.6.5 节点安装成功后，配置kubectl

```console
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

##### 5.6.6 安装网络插件，此处为flannel网络，只需要在其中一台master节点执行

```console
kubectl apply -f https://raw.githubusercontent.com/cherryleo/k8s-apps/master/k8s-flannel/flannel.yaml
```

##### 5.6.7 dashboard安装，只需要在其中一台master节点执行

```console
kubectl apply -f https://raw.githubusercontent.com/cherryleo/k8s-apps/master/k8s-dashboard/kubernetes-dashboard.yaml
kubectl apply -f https://raw.githubusercontent.com/cherryleo/k8s-apps/master/k8s-dashboard/admin-user.yaml
```

##### 5.6.8 修改kube-proxy访问apiserver指向keepalived的虚拟ip

```console
kubectl get configmap -n kube-system kube-proxy -o yaml > kube-proxy-cm.yaml
sed -i "s#server:.*#server: https://${vip}:6443#g" kube-proxy-cm.yaml
kubectl apply -f kube-proxy-cm.yaml --force
kubectl delete pod -n kube-system -l k8s-app=kube-proxy
```

##### 5.6.9 验证集群状态

```console
# kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
master0   Ready     master    18m       v1.9.7
master1   Ready     master    4m        v1.9.7
master2   Ready     master    10m       v1.9.7

# kubectl get pods --all-namespaces -owide
NAMESPACE     NAME                                    READY     STATUS    RESTARTS   AGE       IP           NODE
kube-system   kube-apiserver-master0                  1/1       Running   0          20m       10.250.0.6   master0
kube-system   kube-apiserver-master1                  1/1       Running   0          5m        10.250.0.7   master1
kube-system   kube-apiserver-master2                  1/1       Running   0          11m       10.250.0.8   master2
kube-system   kube-controller-manager-master0         1/1       Running   0          20m       10.250.0.6   master0
kube-system   kube-controller-manager-master1         1/1       Running   0          5m        10.250.0.7   master1
kube-system   kube-controller-manager-master2         1/1       Running   0          12m       10.250.0.8   master2
kube-system   kube-dns-7f5d7475f6-45l9q               3/3       Running   0          20m       10.244.2.3   master1
kube-system   kube-flannel-ds-6sqcb                   1/1       Running   0          3m        10.250.0.7   master1
kube-system   kube-flannel-ds-ck77c                   1/1       Running   0          3m        10.250.0.6   master0
kube-system   kube-flannel-ds-v5rkc                   1/1       Running   0          3m        10.250.0.8   master2
kube-system   kube-proxy-2gw9w                        1/1       Running   0          6m        10.250.0.7   master1
kube-system   kube-proxy-594bh                        1/1       Running   0          12m       10.250.0.8   master2
kube-system   kube-proxy-wggzm                        1/1       Running   0          20m       10.250.0.6   master0
kube-system   kube-scheduler-master0                  1/1       Running   0          20m       10.250.0.6   master0
kube-system   kube-scheduler-master1                  1/1       Running   0          5m        10.250.0.7   master1
kube-system   kube-scheduler-master2                  1/1       Running   0          11m       10.250.0.8   master2
kube-system   kubernetes-dashboard-86b599c599-d87jg   1/1       Running   0          3m        10.244.2.2   master1
```

#### 5.7 安装worker节点，确保worker节点均完整执行5.1 - 5.5步骤

##### 5.7.1 在任一台master节点获取token

```console
# kubeadm token create --print-join-command
kubeadm join --token d36067.c01b90dea2801f9b 10.250.0.7:6443 --discovery-token-ca-cert-hash sha256:e7332846e80fdcc551978c2bd20e4487f1b6b1b27ecbd1edaefedae0789d761c
```

##### 5.7.2 node节点加入集群

```console
kubeadm join --token d36067.c01b90dea2801f9b 10.250.0.7:6443 --discovery-token-ca-cert-hash sha256:e7332846e80fdcc551978c2bd20e4487f1b6b1b27ecbd1edaefedae0789d761c
```

##### 5.7.3 在worker节点设置keepalived环境变量

```console
export vip=10.250.0.10
```

##### 5.7.4 把worker节点kubelet指向keepalived，master节点也需要执行下面指令

```console
sed -i "s#server:.*#server: https://${vip}:6443#g" /etc/kubernetes/kubelet.conf
systemctl restart kubelet
```

##### 5.7.5 在任一台master节点查看集群状态

```console
# kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
master0   Ready     master    43m       v1.9.7
master1   Ready     master    29m       v1.9.7
master2   Ready     master    34m       v1.9.7
worker    Ready     <none>    4m        v1.9.7
```