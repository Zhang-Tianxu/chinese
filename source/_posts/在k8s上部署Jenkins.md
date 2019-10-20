---
title: 在k8s上部署Jenkins
date: 2019-06-26 21:04:50
tags:
  - Jenkins
  - CI/CD
  - k8s
  - 环境配置
  - DevOps
categories:
  - 学习
  - 计算机及软件
  - DevOps
  - Jenkins
top: 9048
---
# 在k8s上部署Jenkins

谷歌在k8s上部署Jenkins的方法，都写的很复杂，各种配置文件一堆，还都失败了！**最终部署成功后发现只需要一句命令就可以了。**

<!--more-->

这个命令就是：

```
kubectl run jenkins --image=jenkinsci/blueocean --port 5000
```

命令解释

1. `--image=jenkinsci/blueocean`用来指定镜像，[`jenkinsci/blueocean` image](https://hub.docker.com/r/jenkinsci/blueocean/)(来自 the [Docker Hub repository](https://hub.docker.com/))。 该镜像包含当前的[长期支持 (LTS) 的Jenkins版本](https://jenkins.io/download) （可以投入使用） ，捆绑了所有Blue Ocean插件和功能。这意味着你不需要单独安装Blue Ocean插件。
2. `--port 端口号`用于指定deployment的端口号，可以随便指定.

通过`kubectl get deployments`可以看到

```
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
jenkins    1/1     1            1           10s
```

显示READY说明deployment创建成功。这是在本地集群已经可以访问，如果想要在外部访问，需要将deployment暴露未service：

```
kubectl expose deployment jenkins --type=NodePort --target-port=8080
```

命令解释：

1. `--type=NodePort`是将service的类型设置为NodePort，这样才能被外部看到
2. `--target-port=8080`是映射（例如“发布”）`jenkinsci/blueocean` 容器的端口8080到主机上的端口8080

这一步完成后执行`kubectl get service`查看：

```
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
jenkins     NodePort    10.107.210.195   <none>        5000:30509/TCP    6m35s
```

可以看出service对外暴露的端口是30509，可以通过`kubectl edit service jenkins`命令将配置中的`- nodePort`修改为你想要暴露出去的端口即可。

浏览器访问得

![image-20190626205811349](https://my-blog-1256501598.cos.ap-beijing.myqcloud.com/github-page/learn/CS/DevOps/jenkins/unlock_jenkins.png)

这里要求***Administrator password***，正常来说是通过`/var/jenkins_home/secrets/initialAdminPassword`来查看，但是因为运行在docker中，所以需要查看docker的log。我们这里更进一步又把docker放到了k8s中，所以要查看jenkins的docker对应的pod的log，方法如下：

1. `kubectl get pods`，找到jenkins对应的pod的NAME，我的对应的名称为jenkins-747ddfbdb6-4msmw

2. `kubectl logs jenkins-747ddfbdb6-4msmw`查看pod的log

3. 在显示的log中就有我们需要的***Administrator password***，格式如下：

   ```
   jenkins.install.SetupWizard init
   INFO:
   
   *************************************************************
   *************************************************************
   *************************************************************
   
   Jenkins initial setup is required. An admin user has been created and a password generated.
   Please use the following password to proceed to installation:
   
   7f54c6c500e242e9bbc804c169c52a3c
   
   This may also be found at: /var/jenkins_home/secrets/initialAdminPassword
   
   *************************************************************
   *************************************************************
   *************************************************************
   ```

   输入上面的password就能进入注册界面：

   ![image-20190626210358399](https://my-blog-1256501598.cos.ap-beijing.myqcloud.com/github-page/learn/CS/DevOps/jenkins/create_admin_user.png)

## 参考文档

Jenkins官方文档：https://jenkins.io/zh/doc/book/installing/#setup-wizard
