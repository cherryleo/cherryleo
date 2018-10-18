# k8s-组件-kubelet

## 1. 简介

Kubelet作为节点代理在每一个节点上运行，主要工作就是创建和管理节点上的Pod（k8s容器组）。通常我们使用一个YAML或者JSON对象描述Pod，Kubelet接收YAML或者JSON数据对Pod进行创建和管理。

Kubelet主要接收来自apiserver组件的数据，可以在启动kubelet时指定主机某个路径下的Pod描述文件，也可以监听某个网络位置上的Pod描述文件进行Pod创建和管理。



## 2. 常用参数

Kubelet支持100多个参数，包括一些特性功能，实验功能等。Kubelet通常作为守护进程或者系统服务运行在节点上。

启动kubelet常用参数是指定集群的配置（如集群域名，集群dns地址，集群pod网络地址等），节点配置（主机名，ip地址等），kubelet配置（数据目录，如何与集群apiserver通信，需要静态启动的pod等）。



## 3. 运行实例

```shell
$ ./kubelet \
--kubeconfig=/etc/kubernetes/kubelet.conf \
--pod-manifest-path=/etc/kubernetes/manifests \
--pod-infra-container-image=my.registry.com/k8s/pause-amd64:3.1 \
--allow-privileged=true \
--network-plugin=cni \
--cni-conf-dir=/etc/cni/net.d \
--cni-bin-dir=/opt/cni/bin \
--cluster-dns=10.96.0.1 \
--cluster-domain=cluster.local \
--cgroup-driver=systemd \
--fail-swap-on=false
```



## 4. 启动参数


```shell
--address                  默认值: 0.0.0.0            kubelet监听地址, 0.0.0.0监听所有接口
--port                     默认值: 10250              kubelet服务端口
--read-only-port           默认值: 10255              kubelet只读端口,针对匿名请求, 设置 0 不启用
--healthz-bind-address     默认值: 127.0.0.1          健康检查监听地址, 0.0.0.0监听所有端口
--healthz-port             默认值: 10248              健康检查监听端口, 设置0不启用
--hostname-override                                  设置hostname, 设置后k8s会使用设置名作为主机名
--container-runtime        默认值: "docker"           容器运行环境, "docker", "rkt"
--enable-server            默认值: true               启用kubelet server
--cluster-dns                                        集群dns地址, ip地址列表
--cluster-domain                                     集群域名
--node-ip                                            节点ip地址, 设置后kubelet使用该ip作为节点地址
--cni-bin-dir              默认值: /opt/cni/bin       cni插件路径
--cni-conf-dir             默认值: /etc/cni/net.d     cni配置文件路径
--network-plugin                                     网络插件名称
--pod-cidr                                           pod ip地址范围
--max-open-files           默认值: 1000000            kubelet进程打开文件数量最大值
--max-pods                 默认值: 110                kubelet运行pod最大数量
--allow-privileged                                   运行容器请求特权模式, true 或 false 
--pod-manifest-path                                  pod描述文件路径, kubelet会根据该路径下的pod描述文件创建pod
--config                                             配置文件, 相对路径或绝对路径, 忽略该参数则使用内置默认值
--register-node            默认值: true Register      自动向apiserver注册
--root-dir                 默认值: "/var/lib/kubelet" kubelet根路径
--anonymous-auth           默认值: true               允许匿名访问kubelet, 匿名用户名/组名(system:anonymous/system:unauthenticated)
--cadvisor-port            默认值: 4194                           本地cAdvisor端口, 设置 0 不启用cAdvisor
--cert-dir                 默认值: "/var/lib/kubelet/pki"         TLS证书路径, 如果设置 --tls-cert-file 和 --tls-private-key-file参数, 该参数会被忽略
--tls-cert-file                                                  x509证书文件
--tls-private-key-file                                           x509私有密钥
--kubeconfig               默认值: "/var/lib/kubelet/kubeconfig"  连接apiserver配置文件路径
--cgroup-driver            默认值: "cgroupfs"                     cgroup驱动, "cgroupfs" 或 "systemd"
--authorization-mode       默认值: "AlwaysAllow"                  授权模式, AlwaysAllow 或 Webhook
--docker-endpoint          默认值: "unix:///var/run/docker.sock"  docker endpoint
--fail-swap-on             默认值: true                           如果swap开启, kubelet会启动失败
--image-gc-high-threshold       默认值: 85            磁盘空间百分比, 磁盘空间超过该比例进行镜像垃圾回收
--image-gc-low-threshold        默认值: 80            磁盘空间百分比, 磁盘空间没超过该比例之前不会进行垃圾回收
--image-pull-progress-deadline  默认值: 1m0s          在设置的时间内无法拉取镜像, 则取消镜像拉取
--minimum-image-ttl-duration    默认值: 2m0s          被垃圾回收镜像最小寿命
--pod-infra-container-image     默认值: "gcr.io/google_containers/pause-amd64:3.1"   pause镜像名称
--container-runtime-endpoint    默认值: "unix:///var/run/dockershim.sock"            可配置远程服务(实验特性)
--containerized                                                                     以容器方式运行kubelet(实验特性)
--authentication-token-webhook Use the TokenReview API to determine authentication for bearer tokens.
--authentication-token-webhook-cache-ttl duration     Default: 2m0s The duration to cache responses from the webhook token authenticator.
--authorization-webhook-cache-authorized-ttl duration     Default: 5m0s The duration to cache 'authorized' responses from the webhook authorizer.
--authorization-webhook-cache-unauthorized-ttl duration     Default: 30s The duration to cache 'unauthorized' responses from the webhook authorizer.
--azure-container-registry-config string Path to the file container Azure container registry configuration information.
--bootstrap-checkpoint-path string Path to to the directory where the checkpoints are stored
--bootstrap-kubeconfig string Path to a kubeconfig file that will be used to get client certificate for kubelet. If the file specified by --kubeconfig does not exist, the bootstrap kubeconfig is used to request a client certificate from the API server. On success, a kubeconfig file referencing the generated client certificate and key is written to the path specified by --kubeconfig. The client certificate and key file will be stored in the directory pointed by --cert-dir.
--cgroup-root string Optional root cgroup to use for pods. This is handled by the container runtime on a best effort basis. Default: '', which means use the container runtime default.
--cgroups-per-qos     Default: true Enable creation of QoS cgroup hierarchy, if true top level QoS and pod cgroups are created.
--chaos-chance float If > 0.0, introduce random client errors and latency. Intended for testing.
--client-ca-file string If set, any request presenting a client certificate signed by one of the authorities in the client-ca-file is authenticated with an identity corresponding to the CommonName of the client certificate.
--cloud-config string The path to the cloud provider configuration file. Empty string for no configuration file.
--cloud-provider string The provider for cloud services. Specify empty string for running with no cloud provider.
--contention-profiling Enable lock contention profiling, if profiling is enabled
--cpu-cfs-quota     Default: true Enable CPU CFS quota enforcement for containers that specify CPU limits
--cpu-manager-policy string     Default: "none" CPU Manager policy to use. Possible values: 'none', 'static'. Default: 'none'
--cpu-manager-reconcile-period NodeStatusUpdateFrequency     Default: 10s CPU Manager reconciliation period. Examples: '10s', or '1m'. If not supplied, defaults to NodeStatusUpdateFrequency
--docker-disable-shared-pid     Default: true The Container Runtime Interface (CRI) defaults to using a shared PID namespace for containers in a pod when running with Docker 1.13.1 or higher. Setting this flag reverts to the previous behavior of isolated PID namespaces. This ability will be removed in a future Kubernetes release.
--dynamic-config-dir string The Kubelet will use this directory for checkpointing downloaded configurations and tracking configuration health. The Kubelet will create this directory if it does not already exist. The path may be absolute or relative; relative paths start at the Kubelet's current working directory. Providing this flag enables dynamic Kubelet configuration. Presently, you must also enable the DynamicKubeletConfig feature gate to pass this flag.
--enable-controller-attach-detach     Default: true Enables the Attach/Detach controller to manage attachment/detachment of volumes scheduled to this node, and disables kubelet from executing any attach/detach operations
--enable-debugging-handlers     Default: true Enables server endpoints for log collection and local running of containers and commands
--enforce-node-allocatable stringSlice     Default: [pods] A comma separated list of levels of node allocatable enforcement to be enforced by kubelet. Acceptable options are 'pods', 'system-reserved' & 'kube-reserved'. If the latter two options are specified, '--system-reserved-cgroup' & '--kube-reserved-cgroup' must also be set respectively. See https://kubernetes.io/docs/tasks/administer-cluster/reserve-compute-resources/ for more details.
--event-burst int32     Default: 10 Maximum size of a bursty event records, temporarily allows event records to burst to this number, while still not exceeding event-qps. Only used if --event-qps > 0
--event-qps int32     Default: 5 If > 0, limit event creations per second to this value. If 0, unlimited.
--eviction-hard mapStringString     Default: imagefs.available<15%!,(MISSING)memory.available<100Mi,nodefs.available<10%!,(MISSING)nodefs.inodesFree<5%!<(MISSING)/td> A set of eviction thresholds (e.g. memory.available<1Gi) that if met would trigger a pod eviction.
--eviction-max-pod-grace-period int32 Maximum allowed grace period (in seconds) to use when terminating pods in response to a soft eviction threshold being met. If negative, defer to pod specified value.
--eviction-minimum-reclaim mapStringString A set of minimum reclaims (e.g. imagefs.available=2Gi) that describes the minimum amount of resource the kubelet will reclaim when performing a pod eviction if that resource is under pressure.
--eviction-pressure-transition-period duration     Default: 5m0s Duration for which the kubelet has to wait before transitioning out of an eviction pressure condition.
--eviction-soft mapStringString A set of eviction thresholds (e.g. memory.available<1.5Gi) that if met over a corresponding grace period would trigger a pod eviction.
--eviction-soft-grace-period mapStringString A set of eviction grace periods (e.g. memory.available=1m30s) that correspond to how long a soft eviction threshold must hold before triggering a pod eviction.
--exit-on-lock-contention Whether kubelet should exit upon lock-file contention.
--experimental-allocatable-ignore-eviction When set to 'true', Hard Eviction Thresholds will be ignored while calculating Node Allocatable. See https://kubernetes.io/docs/tasks/administer-cluster/reserve-compute-resources/ for more details. [default=false]
--experimental-allowed-unsafe-sysctls stringSlice Comma-separated whitelist of unsafe sysctls or unsafe sysctl patterns (ending in *). Use these at your own risk.
--experimental-bootstrap-kubeconfig string deprecated: use --bootstrap-kubeconfig
--experimental-check-node-capabilities-before-mount [Experimental] if set true, the kubelet will check the underlying node for required components (binaries, etc.) before performing the mount
--experimental-kernel-memcg-notification If enabled, the kubelet will integrate with the kernel memcg notification to determine if memory eviction thresholds are crossed rather than polling.
--experimental-mounter-path string [Experimental] Path of mounter binary. Leave empty to use the default mount.
--experimental-qos-reserved mapStringString A set of ResourceName=Percentage (e.g. memory=50%!)(MISSING) pairs that describe how pod resource requests are reserved at the QoS level. Currently only memory is supported. [default=none]
--file-check-frequency duration     默认值: 20s 检查配置文件是否更新时间间隔
--google-json-key string The Google Cloud Platform Service Account JSON Key to use for authentication.
--hairpin-mode string     Default: "promiscuous-bridge" How should the kubelet setup hairpin NAT. This allows endpoints of a Service to loadbalance back to themselves if they should try to access their own Service. Valid values are "promiscuous-bridge", "hairpin-veth" and "none".
--host-ipc-sources stringSlice     Default: [*] Comma-separated list of sources from which the Kubelet allows pods to use the host ipc namespace.
--host-network-sources stringSlice     Default: [*] Comma-separated list of sources from which the Kubelet allows pods to use of host network.
--host-pid-sources stringSlice     Default: [*] Comma-separated list of sources from which the Kubelet allows pods to use the host pid namespace.
--http-check-frequency duration     Default: 20s Duration between checking http for new data
--image-service-endpoint string [Experimental] The endpoint of remote image service. If not specified, it will be the same with container-runtime-endpoint by default. Currently unix socket is supported on Linux, and tcp is supported on windows. Examples:'unix:///var/run/dockershim.sock', 'tcp://localhost:3735'
--iptables-drop-bit int32     Default: 15 The bit of the fwmark space to mark packets for dropping. Must be within the range [0, 31].
--iptables-masquerade-bit int32     Default: 14 The bit of the fwmark space to mark packets for SNAT. Must be within the range [0, 31]. Please match this parameter with corresponding parameter in kube-proxy.
--kube-api-burst int32     Default: 10 Burst to use while talking with kubernetes apiserver
--kube-api-content-type string     Default: "application/vnd.kubernetes.protobuf" Content type of requests sent to apiserver.
--kube-api-qps int32     Default: 5 QPS to use while talking with kubernetes apiserver
--kube-reserved mapStringString A set of ResourceName=ResourceQuantity (e.g. cpu=200m,memory=500Mi,ephemeral-storage=1Gi) pairs that describe resources reserved for kubernetes system components. Currently cpu, memory and local ephemeral storage for root file system are supported. See http://kubernetes.io/docs/user-guide/compute-resources for more detail. [default=none]
--kube-reserved-cgroup string Absolute name of the top level cgroup that is used to manage kubernetes components for which compute resources were reserved via '--kube-reserved' flag. Ex. '/kube-reserved'. [default='']
--kubelet-cgroups string Optional absolute name of cgroups to create and run the Kubelet in.
--lock-file string The path to file for kubelet to use as a lock file.
--make-iptables-util-chains     Default: true If true, kubelet will ensure iptables utility rules are present on host.
--manifest-url string URL for accessing the container manifest
--manifest-url-header --manifest-url-header 'a:hello,b:again,c:world' --manifest-url-header 'b:beautiful' Comma-separated list of HTTP headers to use when accessing the manifest URL. Multiple headers with the same name will be added in the same order provided. This flag can be repeatedly invoked. For example: --manifest-url-header 'a:hello,b:again,c:world' --manifest-url-header 'b:beautiful'
--network-plugin-mtu int32 The MTU to be passed to the network plugin, to override the default. Set to 0 to use the default 1460 MTU.
--node-labels mapStringString Labels to add when registering the node in the cluster. Labels must be key=value pairs separated by ','.
--node-status-update-frequency duration     Default: 10s Specifies how often kubelet posts node status to master. Note: be cautious when changing the constant, it must work with nodeMonitorGracePeriod in nodecontroller.
--oom-score-adj int32     Default: -999 The oom-score-adj value for kubelet process. Values must be within the range [-1000, 1000]
--pods-per-core int32 Number of Pods per core that can run on this Kubelet. The total number of Pods on this Kubelet cannot exceed max-pods, so max-pods will be used if this calculation results in a larger number of Pods allowed on the Kubelet. A value of 0 disables this limit.
--protect-kernel-defaults Default kubelet behaviour for kernel tuning. If set, kubelet errors if any of kernel tunables is different than kubelet defaults.
--provider-id string Unique identifier for identifying the node in a machine database, i.e cloudprovider
--really-crash-for-testing If true, when panics occur crash. Intended for testing.
--register-with-taints []api.Taint Register the node with the given list of taints (comma separated "=:"). No-op if register-node is false.
--registry-burst int32     Default: 10 Maximum size of a bursty pulls, temporarily allows pulls to burst to this number, while still not exceeding registry-qps. Only used if --registry-qps > 0
--registry-qps int32     Default: 5 If > 0, limit registry pull QPS to this value. If 0, unlimited.
--resolv-conf string     Default: "/etc/resolv.conf" Resolver configuration file used as the basis for the container DNS resolution configuration.
--rkt-api-endpoint string     Default: "localhost:15441" The endpoint of the rkt API service to communicate with. Only used if --container-runtime='rkt'.
--rkt-path string Path of rkt binary. Leave empty to use the first rkt in $PATH. Only used if --container-runtime='rkt'.
--rotate-certificates Auto rotate the kubelet client certificates by requesting new certificates from the kube-apiserver when the certificate expiration approaches.
--runonce If true, exit after spawning pods from local manifests or remote urls. Exclusive with --enable-server
--runtime-cgroups string Optional absolute name of cgroups to create and run the runtime in.
--runtime-request-timeout duration     Default: 2m0s Timeout of all runtime requests except long running request - pull, logs, exec and attach. When timeout exceeded, kubelet will cancel the request, throw out an error and retry later.
--seccomp-profile-root string     Default: "/var/lib/kubelet/seccomp" Directory path for seccomp profiles.
--serialize-image-pulls     Default: true Pull images one at a time. We recommend *not* changing the default value on nodes that run docker daemon with version < 1.9 or an Aufs storage backend. Issue #10959 has more details.
--streaming-connection-idle-timeout duration     Default: 4h0m0s Maximum time a streaming connection can be idle before the connection is automatically closed. 0 indicates no timeout. Example: '5m'
--sync-frequency duration     Default: 1m0s Max period between synchronizing running containers and config
--system-cgroups / Optional absolute name of cgroups in which to place all non-kernel processes that are not already inside a cgroup under /. Empty for no container. Rolling back the flag requires a reboot.
--system-reserved mapStringString A set of ResourceName=ResourceQuantity (e.g. cpu=200m,memory=500Mi,ephemeral-storage=1Gi) pairs that describe resources reserved for non-kubernetes components. Currently only cpu and memory are supported. See http://kubernetes.io/docs/user-guide/compute-resources for more detail. [default=none]
--system-reserved-cgroup string Absolute name of the top level cgroup that is used to manage non-kubernetes components for which compute resources were reserved via '--system-reserved' flag. Ex. '/system-reserved'. [default='']
--version version[=true] Print version information and quit
--volume-plugin-dir string     Default: "/usr/libexec/kubernetes/kubelet-plugins/volume/exec/" The full path of the directory in which to search for additional third party volume plugins
--volume-stats-agg-period duration     Default: 1m0s Specifies interval for kubelet to calculate and cache the volume disk usage for all pods and volumes. To disable volume calculations, set to 0.
```



