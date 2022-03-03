---
title: 解决k8s相关组件安装镜像无法拉取
date: 2022-02-18 23:30:26
categories: 
    - [云原生, k8s]
tags: [k8s]
---

### k8s 集群搭建过程相关组件安装

>  在k8s 集群搭建过程中，我们在安装rook存储组件 以及ingress相关组件时，因为要访问https://k8s.gcr.io 去拉取镜像，而在墙内我们时访问不了这个组件的；

所以，我们采用的办法是，从阿里云把相应的镜像版本拉取到本地，然后打tag为官方镜像名称，这样，在拉取镜像的时候就直接从本地拉取，而不用访问https://k8s.gcr.io 去拉取了；

+ rook 镜像

  注意这里的版本要和具体报错的拉取镜像失败的版本一致，相关日志请用命令：

  ```bash
  $ kubectl describe pod ${pod_name} -n rook-ceph
  ```

  查看；

  ```bash
  $ docker pull registry.aliyuncs.com/google_containers/csi-node-driver-registrar:v2.5.0
  $ docker tag registry.aliyuncs.com/google_containers/csi-node-driver-registrar:v2.5.0 k8s.gcr.io/sig-storage/csi-node-driver-registrar:v2.5.0
  $ docker rmi registry.aliyuncs.com/google_containers/csi-node-driver-registrar:v2.5.0
  
  $ docker pull registry.aliyuncs.com/google_containers/csi-provisioner:v3.1.0
  $ docker tag registry.aliyuncs.com/google_containers/csi-provisioner:v3.1.0 k8s.gcr.io/sig-storage/csi-provisioner:v3.1.0
  $ docker rmi registry.aliyuncs.com/google_containers/csi-provisioner:v3.1.0
  
  $ docker pull registry.aliyuncs.com/google_containers/csi-resizer:v1.4.0
  $ docker tag registry.aliyuncs.com/google_containers/csi-resizer:v1.4.0 k8s.gcr.io/sig-storage/csi-resizer:v1.4.0
  $ docker rmi registry.aliyuncs.com/google_containers/csi-resizer:v1.4.0
  
  $ docker pull registry.aliyuncs.com/google_containers/csi-attacher:v3.4.0
  $ docker tag registry.aliyuncs.com/google_containers/csi-attacher:v3.4.0 k8s.gcr.io/sig-storage/csi-attacher:v3.4.0
  $ docker rmi registry.aliyuncs.com/google_containers/csi-attacher:v3.4.0
  
  
  $ docker pull registry.aliyuncs.com/google_containers/csi-snapshotter:v5.0.1
  $ docker tag registry.aliyuncs.com/google_containers/csi-snapshotter:v5.0.1 k8s.gcr.io/sig-storage/csi-snapshotter:v5.0.1
  $ docker rmi registry.aliyuncs.com/google_containers/csi-snapshotter:v5.0.1
  
  ```

  

+ Ingress 镜像（版本获取同rook）

  ```bash
  $ docker pull registry.aliyuncs.com/google_containers/nginx-ingress-controller:v1.0.0
  $ docker tag registry.aliyuncs.com/google_containers/nginx-ingress-controller:v1.0.0 k8s.gcr.io/ingress-nginx/controller:v1.0.0
  $ docker rmi registry.aliyuncs.com/google_containers/nginx-ingress-controller:v1.0.0
  
  $ docker pull registry.aliyuncs.com/google_containers/kube-webhook-certgen:v1.0
  $ docker tag registry.aliyuncs.com/google_containers/kube-webhook-certgen:v1.0 k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.0
  $ docker rmi registry.aliyuncs.com/google_containers/kube-webhook-certgen:v1.0
  ```

  
  
+ fluentd-elasticsearch

  ```bash
  $ docker pull registry.aliyuncs.com/google_containers/fluentd-elasticsearch:1.20
  $ docker tag registry.aliyuncs.com/google_containers/fluentd-elasticsearch:1.20 k8s.gcr.io/fluentd-elasticsearch:1.20
  $ docker rmi registry.aliyuncs.com/google_containers/fluentd-elasticsearch:1.20
  ```

  

  

+ <font color=red>注意 :</font> <font color=orange>上面的命令在每个节点都要执行（比如我的环境 k8s-master、k8s-node1、k8s-node2）</font>

  