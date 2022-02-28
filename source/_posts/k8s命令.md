---
title: k8s命令
date: 2022-02-22 18:37:25
comments: true
categories:
    - [云原生, k8s]
tags: 
    - k8s
---
---

## 1. master节点
   1. **查看node节点加入命令** 
        
        ```bash
        $ kubeadm token create --print-join-command
        ```
        
   2. **查看创建节点日志**  
        ```bash
        $ kubectl describe pods -l app=${pod_name}
              
        $ kubectl describe pod ${pod_name} -n ${name_space}
        ```
   3. **重启k8s集群**
        
        ```bash
        master节点执行：
        $ kubectl drain k8s-node1 --delete-local-data --force --ignore-daemonsets
        $ kubectl delete node k8s-node1
        在相应node节点执行：
        $ kubeadm reset
    $ rm -rf /var/lib/cni/ $HOME/.kube/config /etc/cni/net.d
            
	master节点执行：
	$ kubectl drain k8s-node2 --delete-local-data --force --ignore-daemonsets
	$ kubectl delete node k8s-node2
	在相应node节点执行:
        $ kubeadm reset
        $ rm -rf /var/lib/cni/ $HOME/.kube/config /etc/cni/net.d
        
	删除master节点：
	$ kubectl drain k8s-master --delete-local-data --force --ignore-daemonsets
	$ kubectl delete node k8s-master
	$ kubeadm reset
        $ rm -rf /var/lib/cni/ $HOME/.kube/config /etc/cni/net.d
        ```
   4. **实时查看 Deployment 对象的状态变化**
        ```bash
        $ kubectl rollout status deployment/${deployment_name}
        ```
   5. **修改镜像信息信息**
        
        ```bash
        $ kubectl set image deployment/nginx-deployment nginx=nginx:1.91
        ```
   6. **回滚上一版本**
        
        ```bash
        $ kubectl rollout undo deployment/nginx-deployment
        ```
   7. **查看Deployment 变更对应的版本**
        
        ```bash
        $ kubectl rollout history deployment/nginx-deployment
        ```
   8. **回滚到执行版本**
        
        ```bash
        $ kubectl rollout undo deployment/nginx-deployment --to-revision=2
        ```
   9. **暂停滚动更新**
        
        ```bash
        $ kubectl rollout pause deployment/nginx-deployment
        ```
        <font color=orange>因为我们每修改一次deployment，都会生成一个新的ReplicaSet对象，这样显得有些多余和浪费资源，所以，当我们需要修改deployment的时候，先暂时暂停滚动更新，等到我们一次性修改完成后，再重新恢复即可。</font>
   10. **恢复滚动更新**
       
        ```bash
        $ kubectl rollout resume deployment/nginx-deployment
        ```

## 2. node节点

   1. **重置节点**

      ```bash
      $ kubeadm reset
      删除配置信息
      $ rm -rf /var/lib/cni/ $HOME/.kube/config /etc/cni/net.d
      ```

      

