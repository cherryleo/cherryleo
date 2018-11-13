---
weight: 3000
title: "NFS持久化存储"
date: "2018-11-12"
lastmod: "2018-11-12"
description:  "使用NFS作为Kubernetes持久化存储"
categories:  ["kubernetes"]
tags: ["kubernetes"]
---

## 简介

使用NFS作为k8s集群存储

源项目地址https://github.com/kubernetes-incubator/external-storage/tree/master/nfs-client

pv创建后，存储目录格式为 `${namespace}-${pvcName}-${pvName}`

pv删除后，存储会以 `archieved-${namespace}-${pvcName}-${pvName}` 格式存储于磁盘上

## 关系图

![](https://fileserver-1253732882.cos.ap-chongqing.myqcloud.com/pic/k8s-storage-nfs.png)

## NFS安装 

CentOS7安装NFS

```console
# yum install nfs-utils nfs-utils-lib
```

 启动NFS

```console
# chkconfig nfs on 
# service rpcbind start
# service nfs start
```

配置网络共享目录

```console
# mkdir -p /var/data/k8s-nfs-storage 
# cat << EOF > /etc/exports
/var/data/k8s-nfs-storage           *(rw,sync,no_root_squash,no_subtree_check)
EOF
exportfs -a
```

查看网络共享目录

```console
# showmount -e 10.255.0.110
/var/data/k8s-nfs-storage  *
```

## Kubernetes配置NFS存储 

创建ServiceAccount

```yaml
# serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
```

```console
# kubectl create -f serviceaccount.yaml
```

RBAC认证配置，若未启用RBAC，跳过此步

```yaml
# clusterrole.yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["list", "watch", "create", "update", "patch"]
```

```console
# kubectl create -f clusterrole.yaml
```

```yaml
# clusterrolebinding.yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: default
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
```

```console
# kubectl create -f clusterrolebinding.yaml
```

创建Deployment

```yaml
# deployment.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nfs-client-provisioner
spec:
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: ccr.ccs.tencentyun.com/quay.io/nfs-client-provisioner:v2.0.1
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: 10.255.0.110/nfs  # 名称随便定义
            - name: NFS_SERVER
              value: 10.255.0.110  # FIXME: NFS服务器地址
            - name: NFS_PATH
              value: /var/data/k8s-nfs-storage  # FIXME: NFS网络共享目录
      volumes:
        - name: nfs-client-root
          nfs:
            server: 10.255.0.110  # FIXME: NFS服务器地址
            path: /var/data/k8s-nfs-storage  # FIXME: NFS网络共享目录
```

创建StorageClass

```yaml
# storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-nfs-storage
provisioner: 10.255.0.110/nfs # 此处值需和4.3步骤中deployment.yaml文件中的 PROVISIONER_NAME 一致
```

```console
# kubectl create -f storageclass.yaml
```

## 验证

查看storageclass

```console
[root@10-255-0-196 ~]# kubectl get storageclass
NAME                  PROVISIONER        AGE
managed-nfs-storage   10.255.0.110/nfs   20m
```

创建pvc

```yaml
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: busybox-storage
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: managed-nfs-storage  # 此处值为4.4步骤 storageclass 名称
  resources:
    requests:
      storage: 1Mi
```

```console
# kubectl create -f pvc.yaml 
persistentvolumeclaim "busybox-storage" created
```

创建pod使用pvc

```yaml
# busybox.yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
    - image: ccr.ccs.tencentyun.com/tencentyun/busybox
      name: buxybox
      command:
        - "/bin/sh"
      args:
        - "-c"
        - "touch /var/data/SUCCESS && exit 0 || exit 1"
      volumeMounts:
        - name: nfs-stroage
          mountPath: /var/data
  volumes:
    - name: nfs-stroage
      persistentVolumeClaim:
        claimName: busybox-storage  # 5.2步骤 pvc 名称
```

```console
# kubectl create -f busybox.yaml
```

查看pvc&pv

```console
# kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM                     STORAGECLASS          REASON    AGE
pvc-98b28be3-61c0-11e8-8c8e-0050563e239d   1Mi        RWX            Delete           Bound     default/busybox-storage   managed-nfs-storage             3s
# kubectl get pvc
NAME              STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE
busybox-storage   Bound     pvc-98b28be3-61c0-11e8-8c8e-0050563e239d   1Mi        RWX            managed-nfs-storage   12s
```

查看NFS服务器目录文件

```console
# cd /var/data/k8s-nfs-storage/
# ls
drwxrwxrwx 2 root root 21 5月  27 23:24 default-busybox-storage-pvc-0483c7b8-61c2-11e8-8c8e-0050563e239d
# ls default-busybox-storage-pvc-0483c7b8-61c2-11e8-8c8e-0050563e239d/
SUCCESS
```