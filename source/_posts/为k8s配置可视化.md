---
title: 为k8s配置可视化
tags:
  - k8s
  - kubernetes
  - DevOps
categories:
  - DevOps
date: 2019-06-24 20:53:54
---

# 为k8s配置可视化

K8s dashboard是k8s集群的web-based UI工具，用户可以通过dashboard管理运行在集群上的应用以及集群自身。

安装本身非常容易，只需要下面一条语句就可以

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```

但是再次由于墙的原因，国内执行这条语句会出现`ErrImagePull`的错误。如果你已经执行了，就需要删除这个pod。但是如果直接用`kubectl delete pods <pod name> --namespace=<namespace>`的话，通过`kubectl get pods --all-namespaces`查看会发现它马上就会重新建立。这是因为要直接删除`kubectl delete deployment <deployment name>`。
<!--more-->

所以在国内还得需要镜像

```
docker pull registry.cn-qingdao.aliyuncs.com/wangxiaoke/kubernetes-dashboard-amd64:v1.10.0
docker tag registry.cn-qingdao.aliyuncs.com/wangxiaoke/kubernetes-dashboard-amd64:v1.10.0 k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.0
docker image rm registry.cn-qingdao.aliyuncs.com/wangxiaoke/kubernetes-dashboard-amd64:v1.10.0
```

下载完镜像后执行`kubectl apply -f http://mirror.faasx.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml`就部署完成了，可以通过命令：

```
`kubectl get pods --namespace=kube-system`
```

查看，然后可以通过`kubectl get service --namespace=kube-system`查看dashboard的外网暴露端口。如果发现暴露的端口类型是`ClusterIP`，可以运行`kubectl edit service  kubernetes-dashboard --namespace=kube-system`大概配置文件，找到type，将ClusterIP改成NodePort。同时也可以通过这个命令修改`NodePort`以达到修改暴露端口的目的。

![image-20190624073355080](https://my-blog-1256501598.cos.ap-beijing.myqcloud.com/github-page/learn/CS/DevOps/k8s/kubectl_edit_service.png)

现在在本级上通过访问`localhost:端口`就能访问图形界面了。但是想要在其他机器访问，需要SSL证书，通过https来访问。

1. 生成私钥和证书签名：

   ```
   openssl genrsa -des3 -passout pass:x -out dashboard.pass.key 2048
   openssl rsa -passin pass:x -in dashboard.pass.key -out dashboard.key
   rm dashboard.pass.key
   openssl req -new -key dashboard.key -out dashboard.csr
   ```

2. 生成SSL证书

   ```
   openssl x509 -req -sha256 -days 365 -in dashboard.csr -signkey dashboard.key -out dashboard.crt
   ```

3. 创建dashboard用户

   执行`kubectl create -f dashboard-user-role.yaml`命令，其中`dashboard-user-role.yaml`内容如下：

   ```
   kind: ClusterRoleBinding
   apiVersion: rbac.authorization.k8s.io/v1beta1
   metadata:
     name: admin
     annotations:
       rbac.authorization.kubernetes.io/autoupdate: "true"
   roleRef:
     kind: ClusterRole
     name: cluster-admin
     apiGroup: rbac.authorization.k8s.io
   subjects:
   - kind: ServiceAccount
     name: admin
     namespace: kube-system
   ---
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: admin
     namespace: kube-system
     labels:
       kubernetes.io/cluster-service: "true"
       addonmanager.kubernetes.io/mode: Reconcile
   ```

4. 获取登陆dashboard的token

   ```
   kubectl describe secret/$(kubectl get secret -nkube-system |grep admin|awk '{print $1}') -nkube-system
   ```

   该语句返回的结果中会包含token

5. 登陆

   在其他机器浏览器访问`https://master的ip:暴露端口`，比如我的例子中就是`https://10.0.0.23:30502`。登陆时选择token登陆，粘贴上面命令返回的token就可以成功登陆。

   ![image-20190623211428221](https://my-blog-1256501598.cos.ap-beijing.myqcloud.com/github-page/learn/CS/DevOps/k8s/k8s_dashboard.png)

## 部署官方用例

官方给出了用于测试的用例，如果k8s部署正确的话，可以直接通过下面的两条命令部署：

```
kubectl create namespace sock-shop

kubectl apply -n sock-shop -f "https://github.com/microservices-demo/microservices-demo/blob/master/deploy/kubernetes/complete-demo.yaml?raw=true"

```

`kubectl edit service  front-end --namespace=sock-shop`来编辑服务的配置，主要是端口。

![image-20190624073309186](https://my-blog-1256501598.cos.ap-beijing.myqcloud.com/github-page/learn/CS/DevOps/k8s/kubectl_edit_service_2.png)

部署完成后通过`NodePort:端口`可以直接访问，是一个袜子购物网站，支持购物车等功能：

![image-20190624073545125](https://my-blog-1256501598.cos.ap-beijing.myqcloud.com/github-page/learn/CS/DevOps/k8s/sock_shop.png)



## 参考

1. k8s dashboard官方：https://github.com/kubernetes/dashboard#kubernetes-dashboard
2. K8s heapster官方：https://github.com/kubernetes-retired/heapster
