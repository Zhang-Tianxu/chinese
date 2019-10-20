---
title: 安装kubelet、kubeadm和kubectl
date: 2019-06-24 20:53:21
tags:
  - k8s
  - kubernetes
  - 环境配置
  - DevOps
categories:
  - 学习
  - 计算机及软件
  - DevOps
  - k8s
---
# 通过kubeadm部署k8s cluster

Kubeadm 是一个工具，通过提供 `kubeadm init` 和 `kubeadm join` 来作为创建 Kubernetes 集群的最佳实践“快速路径”。

## 服务器配置

我在配置时一共使用了4台内存为16G，有4个CPU的服务器，它们的配置相同，具体如下：

操作系统：Ubuntu 16.04.6 LTS

内核版本：4.4.0-146-generic

架构：amd64

docker版本：18.9.5/18.9.6

Kubelet版本：v1.15.0

|  主机名  |  内部ip   |       角色       |
| :------: | :-------: | :--------------: |
|  master  | 10.0.0.23 | kubernetes主节点 |
|  node1   | 10.0.0.66 | kubernetes从节点 |
|  node2   | 10.0.0.53 | kubernetes从节点 |
| Slaver03 | 10.0.0.69 | kubernetes从节点 |
<!--more-->

## 硬件要求

1. 一台或多台运行Ubuntu 16.04+, CentOS 7 或 HypriotOS v1.0.1+
2. 每台机器1GB 或者更多内存，太少可能无法运行你的应用。
3. 集群中完整的网络连接，公网或者私网都可以。

## 安装docker

官方提供了安装脚本，在root模式下一条命令即可完成安装：

```
curl -s https://get.docker.com/ | sudo sh
```

在我安装时`https://get.docker.com/`没有被墙，如果被墙，也可以替换为国内镜像。我知道的国内镜像有：

* 阿里云的镜像：http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/intranet
* Daocloud的镜像： https://get.daocloud.io/docker

## 防火墙SELINUX及swap

我查到的教程里都要求关闭防火墙、禁用SELINUX和swap。但是因为我在配置过程中调过很多参数，很多命令都是了一下，所以也不清楚是不是真的必要，现在都写下来，如果没做出了问题，可以参考：

1. 关闭防火墙

   ```
   systemctl disable firewalld.service 
   systemctl stop firewalld.service
   ```

2. 禁用SELINUX

   ```
   setenforce 0
   ```

3. 禁用swap

   这歌是要在所有节点做

   ```
   swapoff -a
   ```

## 安装kubelet、kubeadm 和 kubectl

kubernetes的国外安装其实非常简单，国内安装的主要问题在于kubernetes部件所需的官方镜像在 [http://gcr.io](https://link.zhihu.com/?target=http%3A//gcr.io)(Google Cloud Container Registry)上，很不幸，这个网站被墙了。所以解决了这个问题，国内环境的安装也就简单了。

解决方法也很简单，用国内的一个景象来替换掉官方景象就可以了。以Ubuntu为例，替换方法如下：

```
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb 国内镜像链接 kubernetes-xenial main
EOF
```

目前我知道的国内还能用的景象是阿里的：https://mirrors.aliyun.com/kubernetes/apt/

所以将官方镜像替换为阿里镜像的方法：

```
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
```

替换完整后执行下面两条命令：

```
apt-get update
apt-get install -y kubelet kubeadm kubectl
```

执行第2条命令的时候可能会出现警告：

```
WARNING: The following packages cannot be authenticated!
  cri-tools kubernetes-cni kubelet kubectl kubeadm
E: There were unauthenticated packages and -y was used without --allow-unauthenticated
```

只需要将第二条命令改为`apt-get install -y kubelet kubeadm kubectl --allow-unauthenticated`就可以了。

## 初始化master

初始化master本身也很简单，只需要一条语句`kubeadm init`，但是因为墙的原因，又一次变得复杂。由于官方镜像地址被墙，所以我们需要首先获取所需镜像以及它们的版本。然后从国内镜像站获取。

首先通过`kubeadm config images list`获取完成`kubeadm init`需要的一些镜像，这些镜像本来需要从google的服务器pull，现在我们从国内镜像中下载下来然后更改它们的tag，使得`kubeadm init`知道需要的镜像已经下载完成，然后在执行`kubeadm init`就可以不用pull，直接顺利完成。

下面是我使用`kubeadm config images list`获取的需要的镜像

```
k8s.gcr.io/kube-apiserver:v1.15.0
k8s.gcr.io/kube-controller-manager:v1.15.0
k8s.gcr.io/kube-scheduler:v1.15.0
k8s.gcr.io/kube-proxy:v1.15.0
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.3.10
k8s.gcr.io/coredns:1.3.1
```

国内镜像仓库我知道的有两个，dockerhub的：`mirrorgooglecontainers/` 和阿里的：`registry.cn-hangzhou.aliyuncs.com`

可以直接通过`docker pull 镜像仓库地址/需要的镜像`下载，然后通过`docker tag 原标签 需要的标签`将镜像的标签改为`kubeadm init`需要的标签就可以了，针对上面列出的我需要的镜像，下面这些命令完成了镜像的下载和标签更改：

```
docker pull mirrorgooglecontainers/kube-apiserver:v1.15.0
docker tag mirrorgooglecontainers/kube-apiserver:v1.15.0 k8s.gcr.io/kube-apiserver:v1.15.0
docker pull mirrorgooglecontainers/kube-controller-manager:v1.15.0
docker tag mirrorgooglecontainers/kube-controller-manager:v1.15.0 k8s.gcr.io/kube-controller-manager:v1.15.0
docker pull mirrorgooglecontainers/kube-scheduler:v1.15.0
docker tag mirrorgooglecontainers/kube-scheduler:v1.15.0 k8s.gcr.io/kube-scheduler:v1.15.0
docker pull mirrorgooglecontainers/kube-proxy:v1.15.0
docker tag mirrorgooglecontainers/kube-proxy:v1.15.0 k8s.gcr.io/kube-proxy:v1.15.0
docker pull mirrorgooglecontainers/pause:3.1
docker tag mirrorgooglecontainers/pause:3.1 k8s.gcr.io/pause:3.1
docker pull mirrorgooglecontainers/etcd:3.3.10
docker tag mirrorgooglecontainers/etcd:3.3.10 k8s.gcr.io/etcd:3.3.10
docker pull registry.cn-hangzhou.aliyuncs.com/coredns:1.3.1
docker tag registry.cn-hangzhou.aliyuncs.com/coredns:1.3.1 k8s.gcr.io/coredns:1.3.1
```

上面的命令完成后，`kubeadm init`命令就可以顺利完成了。

接下来的安装变得简单起来，可以参考官网文档中[利用kubeadm 建立一个集群](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)。

步骤就是

1. 初始化master

   就是刚刚说的`kubeadm init`这条语句，不过受下面一步的影响，需要不同参数，所以先不要急着执行，先看完。

2. 安装网络插件

   使得各个pod之间可以通信，有多种选择，每中网络插件支持的架构有所不同，我选了*Weave Net*，*Weave Net* 只能在`amd64`, `arm` 和 `arm64` 上运行。可以通过下面两条命令完成安装：

   ```
   export kubever=$(kubectl version | base64 | tr -d '\n')
   kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$kubever"
   ```

   安装后通过`kubectl get pods --all-namespaces`命令查看各个pod的状态，如果都是Runing，就可以开始添加从节点了。

   注：如果选择*Calico*、*flannel*，为了可以让网络协议正确运行，必须在执行`kubeadm init`时添加参数`--pod-network-cidr=192.168.0.0/16`

3. 添加从节点

   在执行`kubeadm init`的返回结果末尾会给出`kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>`格式的命令，直接复制在自节点root模式下执行就可以完成加入。都加入完成过一分钟返回master通过`kubectl get nodes`查看所有节点的状态。显示应该如下：

   ```
   NAME       STATUS   ROLES    AGE   VERSION
   master     Ready    master   49m   v1.15.0
   node1      Ready    <none>   32m   v1.15.0
   node2      Ready    <none>   47m   v1.15.0
   slaver03   Ready    <none>   17m   v1.15.0
   ```
   
4. 如果需要的话，删除节点

   删除节点之前一定要“排干”节点，也就是将该节点中的pod转移到其他node。

   ```
   kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
   ```

   排干后就可以删除了

   ```
   kubectl delete node <node name>
   ```

   

## 其他一些不重要的点

### 初始化root密码

`sudo passwd root`

### 各节点间免密钥登陆

1. 生成密钥对（如果没有的话）

   执行命令（需要确认的地方全部会车）

   ```
   ssh-keygen -t rsa
   ```

   这样就在`.ssh/`文件夹中生成了`id_rsa`和`id_rsa_pub`密钥对

2. 将公钥加入到要登陆主机的`.ssh/authorized_keys`

   有两种方法

   1. 通过`ssh-copy-id`命令

      ssh-copy-id命令可以把本地的ssh公钥文件安装到远程主机对应的账户下

      达到的功能：

      ssh-copy-id - 将你的公共密钥填充到一个远程机器上的authorized_keys文件中。

      使用方法：

       `ssh-copy-id [-i [identity_file]] [user@]machine`

      比如`ssh-copy-id ubuntu@10.0.0.23`

   2. 远程操作

      可以通过`cat .ssh/id_rsa.pub | ssh ubuntu@10.0.0.23 'cat >> .ssh/authorized_keys'`达到和上述一样的目的。

3. 尝试登陆

   ssh ubuntu@10.0.0.23

### 更改主机名称

可以通过下面这个命令把主机名称修改为`new_host_name`

`hostnamectl --static set-hostname new_host_name`

## 踩过的坑

1. 从节点加入成功，但是通过`kubectl get nodes`查看，master的STATUS处于Ready，但是从节点全部是NotReady。

   这个坑我踩了很久才踩平，尝试过很多方法后发现，不光是kubelet、kubeadm 和 kubectl，连`kubeadm init`需要的那些镜像也需要在所有节点上安装。这样节点在介入后才能正常运行。

2. 执行`kubeadm reset`重置后重新初始化时出错

   重置后会有一些配置残留，主要是etcd集群留下的，删除`/var/lib/etcd`文件夹就可以重新初始化了。

3. coreDNS pod一直处于pending状态

   可能是由于安装**网络插件**之前，`kubeadm init`需要的参数没有添加。

4. 使用kubectl delete pods xxx删除对应的pod,提示删除成功，但是立马又回生成一个

   要先删除对应deployment

## 参考文档

1. k8s官方文档：https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
2. 参考安装教程1:[程序羊](https://www.codesheep.cn/2018/12/27/kubeadm-k8s1-13-1/) :sheep::sheep::sheep::sheep::sheep::sheep:
3. 参考安装教程2:[掘金作者Ethan_cn](https://juejin.im/post/5b8a4536e51d4538c545645c#heading-14):
