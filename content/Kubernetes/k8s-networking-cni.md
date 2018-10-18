# K8S - 网络 - CNI

## 1. 简介

CNI（Container Network Interface）容器网络接口，CNCF项目，一种通用的基于插件的网络解决方案。CNI分为两个库，一个用于定义规范，另一个则实现一些标准插件。

CNI：https://github.com/containernetworking/cni

Plugins：https://github.com/containernetworking/plugins



## 2. CNI规范

CNI规范用于定义如何实现基于CNI的网络插件，比如网络配置使用什么格式，配置中包含哪些字段，插件必须实现的操作等等。

CNI规范要求描述网络配置文件为JSON格式，基于CNI的网络插件必须支持ADD，DEL，GET，VERSION操作。



## 3. CNI插件

CNI官方实现并维护了一些常用插件，如果想自己开发，依赖CNI规范也可以自己写网络插件

- `bridge`：创建桥接，将主机和容器添加到它
- `ipvlan`：在容器中添加一个ipvlan接口
- `loopback`：创建一个lookback接口
- `macvlan`：创建一个新的MAC地址，将所有流量转发到容器
- `ptp`：创建一个veth对
- `vlan`：分配一个vlan设备
- `dhcp`：在主机上运行守护进程以代表容器发出DHCP请求
- `host-local`：维护已分配IP的本地数据库
- `flannel`：使用flannel配置文件生成接口
- `tuning`：调整现有接口的sysctl参数
- `portmap`：一个基于iptables的portmapping插件。将端口从主机的地址空间映射到容器



## 4. 验证

#### 4.1 开始之前 

本次使用centos7系统，确保ip指令已经安装（iproute包），在root用户下进行模拟

#### 4.2 下载CNI插件

```console
# mkdir -p /opt/cni/bin
# wget https://fileserver-1253732882.cos.ap-chongqing.myqcloud.com/cni-plugins-amd64-v0.6.0.tgz -P /opt/cni/bin
# cd /opt/cni/bin/
# tar -zxf cni-plugins-amd64-v0.6.0.tgz
# ls
bridge  cni-plugins-amd64-v0.6.0.tgz  dhcp  flannel  host-local  ipvlan  loopback  macvlan  portmap  ptp  sample  tuning  vlan
```
#### 4.3 创建容器网络命名空间

```console
# contid=12345678
# netnspath=/var/run/netns/$contid
# ip netns add $contid

# 查看网络命名空间^C
# ip netns list
12345678

# 查看网络命名空间的网口，新创建的命名空间应该是空的^C
# ip netns exec 12345678 ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```


#### 4.4 创建网络配置文件

```console
# mkdir -p /etc/cni/net.d
# cat >/etc/cni/net.d/10-bridge.conf <<EOF
{
    "cniVersion": "0.2.0",
    "name": "mynet",
    "type": "bridge",
    "bridge": "cni0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "subnet": "10.22.0.0/16",
        "routes": [
            { "dst": "0.0.0.0/0" }
        ]
    }
}
EOF
# cat >/etc/cni/net.d/99-loopback.conf <<EOF
{
    "cniVersion": "0.2.0",
    "type": "loopback"
}
EOF
```

#### 4.5 定义环境变量 

```console
# 根据CNI规范，CNI插件使用时必须定义的环境变量^C
# export CNI_PATH=/opt/cni/bin
# export CNI_CONTAINERID=$contid
# export CNI_NETNS=$netnspath
# export CNI_COMMAND=ADD
# export CNI_IFNAME=eth0
```

#### 4.6 为容器创建网络接口

##### 4.6.1 先创建一个eth0口连接到bridge网络，使用bridge插件进行创建 

```console
# /opt/cni/bin/bridge </etc/cni/net.d/10-bridge.conf 
{
    "cniVersion": "0.2.0",
    "ip4": {
        "ip": "10.22.0.3/16",
        "gateway": "10.22.0.1",
        "routes": [
            {
                "dst": "0.0.0.0/0",
                "gw": "10.22.0.1"
            }
        ]
    },
    "dns": {}
}
```

##### 4.6.2 查看容器网络接口，此时多出一个eth0接口，ip隶属于bridge网络

```console
# ip netns exec 12345678 ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
3: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP 
    link/ether 0a:58:0a:16:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.22.0.3/16 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::3863:1eff:fe53:8bec/64 scope link 
       valid_lft forever preferred_lft forever
```

##### 4.6.3 创建lookback接口，使用lookback插件进行创建

```console
# /opt/cni/bin/loopback </etc/cni/net.d/99-loopback.conf 
{
    "dns": {}
}
```

##### 4.6.4 查看容器网络接口，此时lookback接口被激活

```console
# ip netns exec 12345678 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
3: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP 
    link/ether 0a:58:0a:16:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.22.0.3/16 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::3863:1eff:fe53:8bec/64 scope link 
       valid_lft forever preferred_lft forever
```

#### 4.7 网络测试

主机多出cni0接口，并且使用bridge网络第一个ip地址，作为bridge网络的网关。容器网络ping主机，ping网关，ping外网都是通的。

```console
# ip netns exec 12345678 ping 10.22.0.1
# ip netns exec 12345678 ping baidu.com
```

关于容器网络的接口如何能通外网的，查看iptables可以看出是创建了snat规则

```console
# iptables -t nat -L
···
Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination         
CNI-23f34ea74dae0bd02dd5ef12  all  --  10.22.0.0/16         anywhere             /* name: "mynet" id: "12345678" */

Chain CNI-23f34ea74dae0bd02dd5ef12 (1 references)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             10.22.0.0/16         /* name: "mynet" id: "12345678" */
MASQUERADE  all  --  anywhere            !base-address.mcast.net/4  /* name: "mynet" id: "12345678" */
```

