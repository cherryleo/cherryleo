# k8s-安全-TLS双向加密认证

## 1. 简介

安全第一步，尝试与k8s建立连接（也就是与apiserver建立连接），使用apiserver的restful接口对k8s集群资源进行查看或者操作，使用tls进行双向加密认证，同时加密通信流量，k8s默认使用的安全端口是6443。

TLS（传输层安全协议）是一种安全协议（前身是SSL），用于加密通信流量和身份认证。身份认证使用非对称加密，非对称加密使用公钥加密只能使用私钥解密，反之亦然。



## 2. TLS认证过程

第一步，客户端生成随机数RNc，服务端生成随机数RNs，相互发送给对方

​	开始之前：

​		客户端（公钥，私钥，RNc），把RNc发送给服务端

​		服务端（公钥，私钥，RNs），把RNs发给客户端

​	结果：

​		客户端（公钥，私钥，RNc，RNs）

​		服务端（公钥，私钥，RNc，RNs）

第二步，证书交换，客户端把证书发给服务端，服务端把证书发给客户端（证书内包含公钥信息）

​	结果：

​		客户端（公钥，私钥，RNc，RNs，服务端公钥）

​		服务端（公钥，私钥，RNc，RNs，客户端公钥）

第三步，计算通信加密密码

​	开始之前：

​		客户端（公钥，私钥，RNc，RNs，服务端公钥，PMS），客户端生成新随机数PMS，并使用服务端公钥加密，然后发送给服务端

​		服务端（公钥，私钥，RNc，RNs，客户端公钥）

​	结果：

​		客户端（公钥，私钥，RNc，RNs，PMS，服务端公钥）

​		服务端（公钥，私钥，RNc，RNs，PMS，客户端公钥），PMS是服务端通过私钥解密出来

​	计算通信加密密码 MS = Calculate(RNc, RNs, PMS)，后续通信使用该密码进行对称加密



## 3. 关于证书

证书通常由权威机构进行签名，做证书的时候需要生成证书请求，然后由ca证书进行签名。可以自己作为ca证书作为权威机构，然后给自己的证书进行签名，也就是制作自签名证书，在公司内部或内网环境经常使用这种方式。



## 4. 使用

生成rootCA.key，然后制作rootCA.crt，制作证书需要填写一些基础信息（国家，地区，公司，部门之类）

```shell
$ openssl genrsa  -out rootCA.key 2048
$ openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -out rootCA.crt
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:Guangdong
Locality Name (eg, city) [Default City]:Shenzhen 
Organization Name (eg, company) [Default Company Ltd]:k8stest
Organizational Unit Name (eg, section) []:tester
Common Name (eg, your name or your server's hostname) []:k8sca 
Email Address []:test@k8s.ca
```

生成k8s-apiserver.key，然后制作证书签名请求文件k8s-apiserver.csr，最后由ca进行签名颁发证书

**非常重要：制作证书签名请求时，Common Name字段一定要正确填写主机名或者主机IP**

```shell
$ openssl genrsa -out k8s-apiserver.key 2048
$ openssl req -new -key k8s-apiserver.key -out k8s-apiserver.csr
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:Guangdong
Locality Name (eg, city) [Default City]:Shenzhen
Organization Name (eg, company) [Default Company Ltd]:k8stest
Organizational Unit Name (eg, section) []:tester
Common Name (eg, your name or your server's hostname) []:192.168.0.104
Email Address []:test@k8s.apiserver

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

使用ca为k8s-apiserver进行签名并颁发证书

```shell
openssl x509 -req -in k8s-apiserver.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out k8s-apiserver.crt -days 500 -sha256
```



## 5. 验证

使用刚才制作的证书启动kube-apiserver，单独启动apiserver仅做tls认证测试

```shell
$ ./kube-apiserver --etcd-servers=http://127.0.0.1:2379 --tls-cert-file=./k8s-apiserver.crt --tls-private-key-file=./k8s-apiserver.key --advertise-address=192.168.0.104 --secure-port=6443 --client-ca-file=./rootCA.crt 
```

在另一台主机访问apiserver，不携带证书进行访问，返回的是401未授权

```shell
curl https://192.168.0.104:6443 -k                                                                    
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {
    
  },
  "status": "Failure",
  "message": "Unauthorized",
  "reason": "Unauthorized",
  "code": 401
}                       
```

直接携带服务端证书进行访问（仅测试，正常情况下应该携带由ca签名的客户端证书），返回api列表

```shell
$ curl https://192.168.0.104:6443 --cacert rootCA.crt --cert k8s-apiserver.crt --key k8s-apiserver.key
{
  "paths": [
    "/api",
    "/api/v1",
    "/apis",
    "/apis/",
    "/apis/admissionregistration.k8s.io",
    "/apis/admissionregistration.k8s.io/v1beta1",
    "/apis/apiextensions.k8s.io",
    "/apis/apiextensions.k8s.io/v1beta1"
    ...
}
```

制作客户端证书并进行访问，客户端证书生成证书请求文件，并使用ca签名并颁发证书

**重要：客户端证书的Common Name需要填写本机IP地址**

```shell
$ openssl genrsa -out client.key 2048 
$ openssl req -new -key client.key -out client.csr
$ openssl x509 -req -in client.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out client.crt -days 500 -sha256
```

使用客户端证书访问apiserver

```shell
curl https://192.168.0.104:6443 --cacert rootCA.crt --cert client.crt --key client.key                  
{
  "paths": [
    "/api",
    "/api/v1",
    "/apis",
    "/apis/",
    "/apis/admissionregistration.k8s.io",
    "/apis/admissionregistration.k8s.io/v1beta1",
    "/apis/apiextensions.k8s.io",
    ...
}
```



## 6. 其他 

可以把rootCA.crt导入浏览器的证书颁发机构，这样通过浏览器访问k8s就不会出现非安全网站提示

![](https://fileserver-1253732882.cos.ap-chongqing.myqcloud.com/pic/k8s-ca.png)

把生成的客户端证书转成pfx格式导入浏览器，则可以通过浏览器访问apiserver

```shell
$ openssl pkcs12 -export -out client.pfx -inkey client.key -in client.crt -certfile rootCA.crt
```

![](https://fileserver-1253732882.cos.ap-chongqing.myqcloud.com/pic/k8s-apiserver-browser.png)