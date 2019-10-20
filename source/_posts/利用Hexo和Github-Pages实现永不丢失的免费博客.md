---
title: 利用Hexo和Github Pages实现永不丢失的免费博客
tags:
  - 博客搭建
  - Hexo
  - NexT
  - 主题
categories:
  - 学习
  - 计算机及软件
  - Web开发
  - 个人博客
date: 2019-02-22 11:12:26
top: -1
---


# 利用Hexo + Github Pages实现永不丢失的免费博客

## 组件安装

[Hexo官方网站](https://hexo.io/zh-cn/)给出的介绍：

> Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 [Markdown](http://daringfireball.net/projects/markdown/)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

Hexo是基于Node.js的，所以安装并使用Hexo之前，必须先安装**Node.js**。

[GitHub Pages官方](https://pages.github.com/)给出的介绍：

> GitHub Pages is a static site hosting service designed to host your personal, organization, or project pages directly from a GitHub repository.
>
> GitHub Pages是一种**静态网站托管服务**，用来托管GitHub仓库中的个人、组织或者项目的网页。

为了将博客部署/备份到github上，我们不仅需要有github账户，还需要下载**Git**。综上我们需要安装的组件有：

1. **Node.js**
2. **Git**
3. **Hexo**

安装方法在网上很容易找到，就不再细说了。

<!--more-->

## 本地搭建Hexo博客

[Hexo官方文档](https://hexo.io/zh-cn/docs/)中包括几种中主要的命令：

1. `hexo init [folder]`
   新建一个网站。如果没有设置 `folder` ，Hexo 默认在目前的文件夹建立网站。
2. `hexo new [layout] <title>`
   新建一篇文章。如果没有设置 `layout` 的话，默认使用 `_config.yml`中的 `default_layout` 参数代替。如果标题包含空格的话，请使用引号括起来。
3. `hexo generate`或者`hexo g`
   生成静态文件。
4. `hexo server`或这`hexo s`
   启动服务器。默认情况下，访问网址为： `http://localhost:4000/`。
5. `hexo deploy`或者`hexo d`
   根据`_config.yml`中的`deploy`设则来部署网站。
6. `hexo clean`
   清除缓存文件 (`db.json`) 和已生成的静态文件 (`public`)。
   在某些情况（尤其是更换主题后），如果发现您对站点的更改无论如何也不生效，您可能需要运行该命令。

那么想要创建一个本地Hexo博客很简单，只需要利用`hexo init`创建一个网站，然后`hexo g`命令生成静态文件，最后使用`hexo s`就可以查看生成的静态网站了。如果想要写新的博客只需使用`hexo new "blog title"`就可以在`/source/_post/`文件夹中生成一个`blog-title.md`文件，编辑这个文件就可以写博客了。博客编辑完成后重新执行`hexo g`就能生成对应的静态文件了。

Hexo有很多的主题可以选择，很多主题简介漂亮、支持功能扩展、自定义样式等。Hexo的外观设置请见本博客另一片篇博文[NexT主题基础外观设置](https://freshmanhaner.github.io/2019/02/22/NexT主题基础外观设置/)。但是这里我们先不介绍相关内容，只需要注意**某些功能的扩展可能会影响到后面会提到的备份。**

## 将博客部署到Github Pages

本地搭建了博客肯定是不够的，我们还得把博客发布出去，这就用到GitHub Pages了，其实就是将本地博客的静态文件上传到github中一个特殊的仓库的master分支中。

如果**你已经有权限往github中上传内容**（至于怎么连接github，网上也有很多[教程](https://www.cnblogs.com/yunquan/p/4862723.html)），想要部署博客到GitHub Pages只需两步：

1. 修改`_config.yml`中的`deploy`为：

   ```
   deploy:
     type: git
     repo: git@github.com:user_name/repository_name.git
     branch: master
   ```

2. 执行命令`npm install hexo-deployer-git --save`

## 备份实现永不丢失

用心写出的博客，绝不能让它由于某些以外丢失。本地的单一本分是不够可靠的，我们可以将源文件上传到GitHub Pages对应仓库的非master分支中。

如果你已经完成了GitHub Pages的部署，那么下面的内容会非常简单：

### 备份

在博客文件夹下执行

```shell
git init
git checkout -b backup_branch_name
git add *
git commit -am "first backup"
git remote add origin git@github.com:user_name/repository_name.git
git push origin backup_branch_name
```

这样就大功告成了！

### 恢复

只要你做了上一步的备份，你就可以在另一台电脑上恢复你的博客，当然这台电脑上要安装上面提到的组件。恢复只需3步：

1. 将GitHub Pages对应仓库克隆到本地
   `git clone git@github.com:user_name/repository_name.git`
2. 进入仓库对应目录，切换到备份分支
   `git checkout -b backup_branch_name`
3. 安装本地hexo，既执行命令
   `npm install hexo --save`

完成上述3步后，就可以使用hexo的各种命令来写博客、部署博客了。

需要注意的是某些扩展功能会导致恢复过程的出错，比如访问次数的统计功能就会使备份过程出错。后面我会详细说明什么功能对博客的恢复会有影响，又如何解决。

## 参考文档

1. hexo官网：https://hexo.io/zh-cn/
2. GitHub Pages官网：https://pages.github.com/
3. 可能漏掉某些参考文档，请作者联系添加引用或删除相关内容。