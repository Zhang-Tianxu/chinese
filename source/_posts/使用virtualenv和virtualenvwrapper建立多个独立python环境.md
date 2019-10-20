---
title: 使用virtualenv和virtualenvwrapper建立多个独立python环境
tags:
  - python
  - virtualenv
  - virtualenvwrapper
  - 虚拟python环境
categories:
  - 学习
  - 计算机及软件
  - 编程语言
  - Python
date: 2019-01-22 11:00:19
top: 9100
---


Python丰富的库是它的优势之一，但是对于我这样的强迫症来说却多少有些不太友好。安装的库越来越多，加上各种库之间的依赖关系。即使能够方便地查看，也会觉得混乱。更不要说还有安装失败的时候，强迫症心里表示很不舒服。
如果你在使用Python，也像我一样是个强迫症，那么救星来了。

![Python](https://www.python.org/static/img/python-logo.png)
<!--more-->  

# virtualenv
[virtualenv](https://pypi.org/project/virtualenv/)是建立独立Python环境的工具，独立的Python环境在实际中是非常必要的。比如你的某个软件依赖某个库的版本1，另一个软件依赖这个库的版本2，如果你把这个库装在同一个python环境中，你很可能把某些你不想升级的库升级了，从而导致一些软件的失效。  
搞清楚了有什么用，下面我们来安装吧，这里以Windows环境为例。有了神器pip的帮助，virtualenv的安装很简单，进入cmd，然后只需`pip install virtualenv`这条命令就能安装成功。如果遇到问题，详细解决请参见[官方文件](https://virtualenv.pypa.io/en/latest/)。  
安装简单，使用起来还很简答。进入你需要防止独立Python环境的目录下，执行`virtualenv your_env_name`Python虚拟环境就能创建成功。如果现在就想使用这个环境，只需要`cd your_env_name/Scripts`，找到*Scripts*目录下的*avtivate.bat*这个脚本文件，执行就能进入虚拟环境。你可以像在真实环境下一样通过`pip list`查看已经看装的库，或者安装/卸载你需要的库，我就不再多说了。当你想要退出的时候，也只需要运行*deactivate.bat*文件就可以了。  
这些虚拟环境可以任意放置，而每次使用都要进入虚拟环境目录下来运行脚本文件。管理起来，使用起来未免还是有些麻烦，对晚期强迫症来说还是不够，如果有工具能够帮助我们管理虚拟环境，在任意位置都能使用就好了。  
# virtualenvwrapper
## 简介
[virtualenvwrapper](https://pypi.org/project/virtualenvwrapper/)就是你需要的管理软件，看名字就可以看出，它主要是对virtualenv的一些功能做了封装，方便我们使用。官方列出的功能如下：
> 1. Organizes all of your virtual environments in one place.
> 2. Wrappers for creating, copying and deleting environments, including user-configurable hooks.
> 3. Use a single command to switch between environments.
> 4. Tab completion for commands that take a virtual environment as argument.
> 5. User-configurable hooks for all operations.
> 6. Plugin system for more creating sharable extensions.

我就不翻译了，反正你知道它很有用就是了。下面介绍安装和简单的使用，还是以Windows环境为例。安装还是依靠咱们的*pip*神器，`pip install virtualenvwrapper-win`（linux下是`pip install virtualenvwrapper`就可以了）。安装成功后的简单使用如下：
## 使用
### 新建virtualenv
`mkvirtualenv your_env_name`，所有的Python环境会被集中放置在同一目录下，命令返回的内容会告诉你在哪里。
### 删除虚拟环境
`rmvirtualenv your_env_name`，用于删除某个指定的虚拟环境。
### 查看所有Python环境
`workon`
### 进入某个Python环境
`workon the_env_you_want_enter`
### 退出当前Python环境
`deactivate`
virtualenvwrapper的功能当然不知于此，更多功能等你探索。

## 参考文档

1. https://virtualenv.pypa.io/en/latest/
2. https://virtualenvwrapper.readthedocs.io/en/latest/

