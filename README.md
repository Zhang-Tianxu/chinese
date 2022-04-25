## 如何从头开始
1. clone本repo
2. 安装依赖：
	1. 安装npm
	2. hexo:`npm install hexo-cli -g`
3. 执行`npm install`/`cnpm install`
4. 新建/修改文章
5. push到master分支
6. github Action/travis会自动完成（如果自动化失效，可以手动将public文件夹push到gh-pages分支）

## 如何新建文章
hexo new [文章名]或者直接在`/source/_posts/`中新建`.md`的文件

```
---
title: 文章标题
tags:
    - tag1
    - tag2
categories:
    - 一级目录
    - 二级目录
    - 三级目录
    - ……
[mathjax: 是否启用数学共识，true表示使用]
[top: 一个数字，越大文章排序越是考顶部]
date: 2021-1-1 17:30:21
[comments: false。关闭该文章的评论功能，默认打开评论功能]
---
直接展示的文章内容
<!--more-->
需要点击`read more`查看的文章内容
```
### 如何插入置顶大小的图片
`{% img 图片类名 图片链接 图片宽 图片高 '"图片标题" "加载失败时显示的文字"' %}`
比如
{% img my_img https://tva1.sinaimg.cn/large/008eGmZEgy1gocm3k2n6uj30ia0lujua.jpg 200 100 '"网易云音乐" "加载失败"' %}


