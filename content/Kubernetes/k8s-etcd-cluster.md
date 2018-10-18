# Kubernetes etcd cluster

## 1. 简介

etcd集群部署



## 2. 环境

| 角色  | 系统    | IP           |
| ----- | ------- | ------------ |
| etcd0 | centos7 | 10.255.0.101 |
| etcd1 | centos7 | 10.255.0.159 |
| etcd2 | centos7 | 10.255.0.196 |



## 3. etcd安装

### 3.1 下载地址

官方下载地址：<https://github.com/coreos/etcd/releases/>

内网下载地址: <http://e-proxy.yfb.sunline.cn/repos/offline/k8s/1.9.1/etcd-v3.2.13-linux-amd64.tar.gz> / [http://proxy.yfb.sunline.cn/repos/offline/k8s/1.9.1/etcd-v3.2.13-linux-amd64.tar.gz](http://e-proxy.yfb.sunline.cn/repos/offline/k8s/1.9.1/etcd-v3.2.13-linux-amd64.tar.gz)

### 3.2 解压安装

```
[root@master0 ~]# tar zxvf etcd-v3.2.14-linux-amd64.tar.gz
[root@master0 ~]# cd etcd-v3.2.13-linux-amd64/
[root@master0 ~]# cp etcd etcdctl /usr/local/bin
```



## 4. 安装步骤

### 4.1 设置环境变量

```
# master0, 注意修改current_host值
[root@master0 ~]# export etcd0=10.255.0.101
[root@master0 ~]# export etcd1=10.255.0.159
[root@master0 ~]# export etcd2=10.255.0.196
[root@master0 ~]# export current_host=etcd0

# master1, 注意修改current_host值
[root@master1 ~]# export etcd0=10.255.0.101
[root@master1 ~]# export etcd1=10.255.0.159
[root@master1 ~]# export etcd2=10.255.0.196
[root@master1 ~]# export current_host=etcd1

# master2, 注意修改current_host值
[root@master2 ~]# export etcd0=10.255.0.101
[root@master2 ~]# export etcd1=10.255.0.159
[root@master2 ~]# export etcd2=10.255.0.196
[root@master2 ~]# export current_host=etcd2
```

### 4.2 创建service，三台主机均要执行

```
cat >/etc/systemd/system/etcd.service <<EOF
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

### 4.3 启动service

```
[root@master0 ~]# systemctl daemon-reload
[root@master0 ~]# systemctl start etcd

[root@master1 ~]# systemctl daemon-reload
[root@master1 ~]# systemctl start etcd

[root@master2 ~]# systemctl daemon-reload
[root@master2 ~]# systemctl start etcd
```



## 5. 测试

```
[root@master0 etcd-v3.1.10-linux-amd64]# etcdctl cluster-health
member 1acb61ea30f8ebc is healthy: got healthy result from http://10.255.0.159:2379
member 82ae338261b73f3 is healthy: got healthy result from http://10.255.0.101:2379
member 6cacdca73fedbc7f is healthy: got healthy result from http://10.255.0.196:2379
cluster is healthy

[root@master0 etcd-v3.1.10-linux-amd64]# etcdctl member list
1acb61ea30f8ebc: name=etcd1 peerURLs=http://10.255.0.159:2380 clientURLs=http://10.255.0.159:2379 isLeader=true
82ae338261b73f3: name=etcd0 peerURLs=http://10.255.0.101:2380 clientURLs=http://10.255.0.101:2379 isLeader=false
6cacdca73fedbc7f: name=etcd2 peerURLs=http://10.255.0.196:2380 clientURLs=http://10.255.0.196:2379 isLeader=false
```

