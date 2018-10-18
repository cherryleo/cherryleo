# k8s-组件-apiserver

## 1. 简介

K8s-apiserver组件是k8s集群核心组件，所有其他组件均需要与apiserver进行交互。apiserver提供rest服务，并对k8s资源进行操作。

通常，与k8s集群进行交互即是与apiserver进行交互。apiserver也为集群提供授权和认证功能。



## 2. 常用参数

apiserver支持超过100个参数，常用参数用来配置集群的认证模式，绑定的ip地址和端口，客户端证书，etcd地址等。

通过apiserver启动参数指定k8s集群提供的service地址范围，nodeport端口范围。



## 3. 运行实例

启动apiserver，必须要提供etcd地址，如果没有etcd，则apiserver无法启动

```shell
$ ./kube-apiserver \
--etcd-servers=http://127.0.0.1:2379 \
--authorization-mode=Node,RBAC \
--advertise-address=10.255.0.168 \
--secure-port=6443 \
--insecure-port=0 \
--service-cluster-ip-range=10.96.0.0/12 \
--allow-privileged=true \
--client-ca-file=/etc/kubernetes/pki/ca.crt \
--tls-private-key-file=/etc/kubernetes/pki/apiserver.key \
--tls-cert-file=/etc/kubernetes/pki/apiserver.crt \
--kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname  \
--admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota
```



## 4. 启动参数

```shell
--etcd-servers                                          etcd服务端列表(scheme://ip:port,scheme://ip:port)
--etcd-cafile                                           etcd ca文件
--etcd-certfile                                         etcd ssl通信证书
--etcd-keyfile                                          etcl ssl通信key
--etcd-prefix           默认值: "/registry"              etcd数据存储路径
--bind-address          默认值: 0.0.0.0                  安全端口监听地址, 不设置则使用0.0.0.0监听所有端口
--advertise-address                                     apiserver在集群中的通信IP地址, 若不设置则使用--bind-address参数值, 若--bind-address参数为空, 使用主机默认接口ip
--secure-port           默认值: 6443                     HTTPS端口, 设置0不启用
--basic-auth-file                                       http基础认证文件
--cert-dir              默认值: "/var/run/kubernetes"    TLS证书文件夹路径 若已设置--tls-cert-file 和 --tls-private-key-file参数, 忽略该参数
--client-ca-file                                        客户端ca证书文件 对客户端证书的commonname校验
--admission-control-config-file                         准入控制配置文件
--target-ram-mb                                         apiserver内存限制值, MB
--tls-cert-file                                         tls证书文件
--tls-private-key-file                                  tls私有key文件
--token-auth-file                                       token认证文件, 安全端口使用token进行认证
--service-cluster-ip-range       默认值: 10.0.0.0/24     集群service地址范围, 不能和pod地址范围冲突
--service-node-port-range        默认值: 30000-32767     NodePort范围
--storage-backend                默认值: etcd3           数据存储'etcd3' 或 'etcd2'
--allow-privileged               默认值: false           容器特权模式
--anonymous-auth                 默认值: true            允许匿名请求(匿名用户名/组名 system:anonymous/system:unauthenticated)
--apiserver-count                默认值: 1               集群中apiserver的数量
--authorization-mode             默认值: "AlwaysAllow"   安全端口授权模式列表: AlwaysAllow,AlwaysDeny,ABAC,Webhook,RBAC,Node
--authorization-policy-file                             授权模式为--authorization-mode=ABAC的csv格式授权策略文件
--enable-swagger-ui                                     启用swagger-ui, 路径/swagger-ui
--endpoint-reconciler-type       默认值: "master-count"  值(master-count, lease, none)
--kubelet-certificate-authority                         客户端证书认证文件
--kubelet-client-certificate                            客户端tls证书文件
--kubelet-client-key                                    客户端tls key文件
--kubelet-https                  默认值: true            使用https与kubelet建立连接
--kubelet-timeout duration       默认值: 5s              kubelet操作超时时间
--audit-log-batch-buffer-size int     Default: 10000 The size of the buffer to store events before batching and writing. Only used in batch mode.
--audit-log-batch-max-size int     Default: 400 The maximum size of a batch. Only used in batch mode.
--audit-log-batch-max-wait duration     Default: 30s The amount of time to wait before force writing the batch that hadn't reached the max size. Only used in batch mode.
--audit-log-batch-throttle-burst int     Default: 15 Maximum number of requests sent at the same moment if ThrottleQPS was not utilized before. Only used in batch mode.
--audit-log-batch-throttle-enable Whether batching throttling is enabled. Only used in batch mode.
--audit-log-batch-throttle-qps float32     Default: 10 Maximum average number of batches per second. Only used in batch mode.
--audit-log-format string     Default: "json" Format of saved audits. "legacy" indicates 1-line text format for each event. "json" indicates structured json format. Requires the 'AdvancedAuditing' feature gate. Known formats are legacy,json.
--audit-log-maxage int The maximum number of days to retain old audit log files based on the timestamp encoded in their filename.
--audit-log-maxbackup int The maximum number of old audit log files to retain.
--audit-log-maxsize int The maximum size in megabytes of the audit log file before it gets rotated.
--audit-log-mode string     Default: "blocking" Strategy for sending audit events. Blocking indicates sending events should block server responses. Batch causes the backend to buffer and write events asynchronously. Known modes are batch,blocking.
--audit-log-path string If set, all requests coming to the apiserver will be logged to this file. '-' means standard out.
--audit-policy-file string Path to the file that defines the audit policy configuration. Requires the 'AdvancedAuditing' feature gate. With AdvancedAuditing, a profile is required to enable auditing.
--audit-webhook-batch-buffer-size int     Default: 10000 The size of the buffer to store events before batching and writing. Only used in batch mode.
--audit-webhook-batch-max-size int     Default: 400 The maximum size of a batch. Only used in batch mode.
--audit-webhook-batch-max-wait duration     Default: 30s The amount of time to wait before force writing the batch that hadn't reached the max size. Only used in batch mode.
--audit-webhook-batch-throttle-burst int     Default: 15 Maximum number of requests sent at the same moment if ThrottleQPS was not utilized before. Only used in batch mode.
--audit-webhook-batch-throttle-enable     Default: true Whether batching throttling is enabled. Only used in batch mode.
--audit-webhook-batch-throttle-qps float32     Default: 10 Maximum average number of batches per second. Only used in batch mode.
--audit-webhook-config-file string Path to a kubeconfig formatted file that defines the audit webhook configuration. Requires the 'AdvancedAuditing' feature gate.
--audit-webhook-initial-backoff duration     Default: 10s The amount of time to wait before retrying the first failed request.
--audit-webhook-mode string     Default: "batch" Strategy for sending audit events. Blocking indicates sending events should block server responses. Batch causes the backend to buffer and write events asynchronously. Known modes are batch,blocking.
--authentication-token-webhook-cache-ttl duration     Default: 2m0s The duration to cache responses from the webhook token authenticator.
--authentication-token-webhook-config-file string File with webhook configuration for token authentication in kubeconfig format. The API server will query the remote service to determine authentication for bearer tokens.
--authorization-webhook-cache-authorized-ttl duration     Default: 5m0s The duration to cache 'authorized' responses from the webhook authorizer.
--authorization-webhook-cache-unauthorized-ttl duration     Default: 30s The duration to cache 'unauthorized' responses from the webhook authorizer.
--authorization-webhook-config-file string File with webhook configuration in kubeconfig format, used with --authorization-mode=Webhook. The API server will query the remote service to determine access on the API server's secure port.
--azure-container-registry-config string Path to the file containing Azure container registry configuration information.
--cloud-config string The path to the cloud provider configuration file. Empty string for no configuration file.
--cloud-provider string The provider for cloud services. Empty string for no provider.
--contention-profiling Enable lock contention profiling, if profiling is enabled
--cors-allowed-origins stringSlice List of allowed origins for CORS, comma separated. An allowed origin can be a regular expression to support subdomain matching. If this list is empty CORS will not be enabled.
--default-watch-cache-size int     Default: 100 Default watch cache size. If zero, watch cache will be disabled for resources that do not have a default watch size set.
--delete-collection-workers int     Default: 1 Number of workers spawned for DeleteCollection call. These are used to speed up namespace cleanup.
--deserialization-cache-size int Number of deserialized json objects to cache in memory.
--disable-admission-plugins stringSlice admission plugins that should be disabled although they are in the default enabled plugins list. Comma-delimited list of admission plugins: AlwaysAdmit, AlwaysDeny, AlwaysPullImages, DefaultStorageClass, DefaultTolerationSeconds, DenyEscalatingExec, DenyExecOnPrivileged, EventRateLimit, ExtendedResourceToleration, ImagePolicyWebhook, InitialResources, Initializers, LimitPodHardAntiAffinityTopology, LimitRanger, MutatingAdmissionWebhook, NamespaceAutoProvision, NamespaceExists, NamespaceLifecycle, NodeRestriction, OwnerReferencesPermissionEnforcement, PersistentVolumeClaimResize, PersistentVolumeLabel, PodNodeSelector, PodPreset, PodSecurityPolicy, PodTolerationRestriction, Priority, ResourceQuota, SecurityContextDeny, ServiceAccount, StorageObjectInUseProtection, ValidatingAdmissionWebhook. The order of plugins in this flag does not matter.
--enable-admission-plugins stringSlice admission plugins that should be enabled in addition to default enabled ones. Comma-delimited list of admission plugins: AlwaysAdmit, AlwaysDeny, AlwaysPullImages, DefaultStorageClass, DefaultTolerationSeconds, DenyEscalatingExec, DenyExecOnPrivileged, EventRateLimit, ExtendedResourceToleration, ImagePolicyWebhook, InitialResources, Initializers, LimitPodHardAntiAffinityTopology, LimitRanger, MutatingAdmissionWebhook, NamespaceAutoProvision, NamespaceExists, NamespaceLifecycle, NodeRestriction, OwnerReferencesPermissionEnforcement, PersistentVolumeClaimResize, PersistentVolumeLabel, PodNodeSelector, PodPreset, PodSecurityPolicy, PodTolerationRestriction, Priority, ResourceQuota, SecurityContextDeny, ServiceAccount, StorageObjectInUseProtection, ValidatingAdmissionWebhook. The order of plugins in this flag does not matter.
--enable-aggregator-routing Turns on aggregator routing requests to endoints IP rather than cluster IP.
--enable-bootstrap-token-auth Enable to allow secrets of type 'bootstrap.kubernetes.io/token' in the 'kube-system' namespace to be used for TLS bootstrapping authentication.
--enable-garbage-collector     Default: true Enables the generic garbage collector. MUST be synced with the corresponding flag of the kube-controller-manager.
--enable-logs-handler     Default: true If true, install a /logs handler for the apiserver logs.
--etcd-compaction-interval duration     Default: 5m0s The interval of compaction requests. If 0, the compaction request from apiserver is disabled.
--etcd-count-metric-poll-period duration     Default: 1m0s Frequency of polling etcd for number of resources per type. 0 disables the metric collection.
--etcd-servers-overrides stringSlice Per-resource etcd servers overrides, comma separated. The individual override format: group/resource#servers, where servers are http://ip:port, semicolon separated.
--event-ttl duration     Default: 1h0m0s Amount of time to retain events.
--experimental-encryption-provider-config string The file containing configuration for encryption providers to be used for storing secrets in etcd
--external-hostname string The hostname to use when generating externalized URLs for this master (e.g. Swagger API Docs).
-h, --help help for kube-apiserver
--http2-max-streams-per-connection int The limit that the server gives to clients for the maximum number of streams in an HTTP/2 connection. Zero means to use golang's default.
--kubelet-preferred-address-types stringSlice     Default: [Hostname,InternalDNS,InternalIP,ExternalDNS,ExternalIP] List of the preferred NodeAddressTypes to use for kubelet connections.
--kubelet-read-only-port uint     Default: 10255 DEPRECATED: kubelet port.
--kubernetes-service-node-port int If non-zero, the Kubernetes master service (which apiserver creates/maintains) will be of type NodePort, using this as the value of the port. If zero, the Kubernetes master service will be of type ClusterIP.
--log-flush-frequency duration     Default: 5s Maximum number of seconds between log flushes
--master-service-namespace string     Default: "default" DEPRECATED: the namespace from which the kubernetes master services should be injected into pods.
--max-connection-bytes-per-sec int If non-zero, throttle each user connection to this number of bytes/sec. Currently only applies to long-running requests.
--max-mutating-requests-inflight int     Default: 200 The maximum number of mutating requests in flight at a given time. When the server exceeds this, it rejects requests. Zero for no limit.
--max-requests-inflight int     Default: 400 The maximum number of non-mutating requests in flight at a given time. When the server exceeds this, it rejects requests. Zero for no limit.
--min-request-timeout int     Default: 1800 An optional field indicating the minimum number of seconds a handler must keep a request open before timing it out. Currently only honored by the watch request handler, which picks a randomized value above this number as the connection timeout, to spread out load.
--oidc-ca-file string If set, the OpenID server's certificate will be verified by one of the authorities in the oidc-ca-file, otherwise the host's root CA set will be used.
--oidc-client-id string The client ID for the OpenID Connect client, must be set if oidc-issuer-url is set.
--oidc-groups-claim string If provided, the name of a custom OpenID Connect claim for specifying user groups. The claim value is expected to be a string or array of strings. This flag is experimental, please see the authentication documentation for further details.
--oidc-groups-prefix string If provided, all groups will be prefixed with this value to prevent conflicts with other authentication strategies.
--oidc-issuer-url string The URL of the OpenID issuer, only HTTPS scheme will be accepted. If set, it will be used to verify the OIDC JSON Web Token (JWT).
--oidc-signing-algs stringSlice     Default: [RS256] Comma-separated list of allowed JOSE asymmetric signing algorithms. JWTs with a 'alg' header value not in this list will be rejected. Values are defined by RFC 7518 https://tools.ietf.org/html/rfc7518#section-3.1.
--oidc-username-claim string     Default: "sub" The OpenID claim to use as the user name. Note that claims other than the default ('sub') is not guaranteed to be unique and immutable. This flag is experimental, please see the authentication documentation for further details.
--oidc-username-prefix string If provided, all usernames will be prefixed with this value. If not provided, username claims other than 'email' are prefixed by the issuer URL to avoid clashes. To skip any prefixing, provide the value '-'.
--profiling     Default: true Enable profiling via web interface host:port/debug/pprof/
--proxy-client-cert-file string Client certificate used to prove the identity of the aggregator or kube-apiserver when it must call out during a request. This includes proxying requests to a user api-server and calling out to webhook admission plugins. It is expected that this cert includes a signature from the CA in the --requestheader-client-ca-file flag. That CA is published in the 'extension-apiserver-authentication' configmap in the kube-system namespace. Components receiving calls from kube-aggregator should use that CA to perform their half of the mutual TLS verification.
--proxy-client-key-file string Private key for the client certificate used to prove the identity of the aggregator or kube-apiserver when it must call out during a request. This includes proxying requests to a user api-server and calling out to webhook admission plugins.
--repair-malformed-updates     Default: true If true, server will do its best to fix the update request to pass the validation, e.g., setting empty UID in update request to its existing value. This flag can be turned off after we fix all the clients that send malformed updates.
--request-timeout duration     Default: 1m0s An optional field indicating the duration a handler must keep a request open before timing it out. This is the default request timeout for requests but may be overridden by flags such as --min-request-timeout for specific types of requests.
--requestheader-allowed-names stringSlice List of client certificate common names to allow to provide usernames in headers specified by --requestheader-username-headers. If empty, any client certificate validated by the authorities in --requestheader-client-ca-file is allowed.
--requestheader-client-ca-file string Root certificate bundle to use to verify client certificates on incoming requests before trusting usernames in headers specified by --requestheader-username-headers
--requestheader-extra-headers-prefix stringSlice List of request header prefixes to inspect. X-Remote-Extra- is suggested.
--requestheader-group-headers stringSlice List of request headers to inspect for groups. X-Remote-Group is suggested.
--requestheader-username-headers stringSlice List of request headers to inspect for usernames. X-Remote-User is common.
--runtime-config mapStringString A set of key=value pairs that describe runtime configuration that may be passed to apiserver. <group>/<version> (or <version> for the core group) key can be used to turn on/off specific api versions. <group>/<version>/<resource> (or <version>/<resource> for the core group) can be used to turn on/off specific resources. api/all and api/legacy are special keys to control all and legacy api versions respectively.
--service-account-api-audiences stringSlice Identifiers of the API. The service account token authenticator will validate that tokens used against the API are bound to at least one of these audiences.
--service-account-issuer string Identifier of the service account token issuer. The issuer will assert this identifier in "iss" claim of issued tokens. This value is a string or URI.
--service-account-key-file stringArray File containing PEM-encoded x509 RSA or ECDSA private or public keys, used to verify ServiceAccount tokens. The specified file can contain multiple keys, and the flag can be specified multiple times with different files. If unspecified, --tls-private-key-file is used. Must be specified when --service-account-signing-key is provided
--service-account-lookup     Default: true If true, validate ServiceAccount tokens exist in etcd as part of authentication.
--service-account-signing-key-file string Path to the file that contains the current private key of the service account token issuer. The issuer will sign issued ID tokens with this private key. (Ignored unless alpha TokenRequest is enabled
--storage-media-type string     Default: "application/vnd.kubernetes.protobuf" The media type to use to store objects in storage. Some resources or storage backends may only support a specific media type and will ignore this setting.
--storage-versions string     Default: "admission.k8s.io/v1beta1,
--tls-cipher-suites stringSlice Comma-separated list of cipher suites for the server. Values are from tls package constants (https://golang.org/pkg/crypto/tls/#pkg-constants). If omitted, the default Go cipher suites will be used
--tls-min-version string Minimum TLS version supported. Value must match version names from https://golang.org/pkg/crypto/tls/#pkg-constants.
--tls-sni-cert-key namedCertKey     Default: [] A pair of x509 certificate and private key file paths, optionally suffixed with a list of domain patterns which are fully qualified domain names, possibly with prefixed wildcard segments. If no domain patterns are provided, the names of the certificate are extracted. Non-wildcard matches trump over wildcard matches, explicit domain patterns trump over extracted names. For multiple key/certificate pairs, use the --tls-sni-cert-key multiple times. Examples: "example.crt,example.key" or "foo.crt,foo.key:*.foo.com,foo.com".
--version version[=true] Print version information and quit
--watch-cache     Default: true Enable watch caching in the apiserver
--watch-cache-sizes stringSlice List of watch cache sizes for every resource (pods, nodes, etc.), comma separated. The individual override format: resource[.group]#size, where resource is lowercase plural (no version), group is optional, and size is a number. It takes effect when watch-cache is enabled. Some resources (replicationcontrollers, endpoints, nodes, pods, services, apiservices.apiregistration.k8s.io) have system defaults set by heuristics, others default to default-watch-cache-size
```

