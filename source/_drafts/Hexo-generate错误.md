---
title: Hexo generate错误
date: 2019-02-22 11:06:26
tags:
	- 博客搭建
	- Hexo
	- NexT
	- 主题
	- bug
categories:
	- 学习
	- 计算机及软件
	- Web开发
	- 个人博客
---

# Hexo generate 错误
某次在用hexo写博客的时候，生成静态文件时出现以下错误，删除新写的博客后错误就没了。

```bash
INFO  Start processing
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Template render error: (unknown path)
  parseIf: expected elif, else, or endif, got end of file
```
查找原因后发现是因为新写的博客中出现了“\{ \{\}\}或者\{ % %\}”，影响了hexo模板的转移。