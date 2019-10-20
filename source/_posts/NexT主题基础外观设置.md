---
title: NexT主题基础外观设置
tags:
  - 博客搭建
  - Hexo
  - NexT
  - 主题
categories:
  - 专业学习
  - 前端
  - Web
date: 2019-02-22 10:45:15
---

# NexT主题基础外观设置




## 更换语言

NexT支持多种语言：

| 语言         | 代码                 | 设定示例                            |
| ------------ | -------------------- | ----------------------------------- |
| English      | `en`                 | `language: en`                      |
| 简体中文     | `zh-Hans`            | `language: zh-Hans`                 |
| Français     | `fr-FR`              | `language: fr-FR`                   |
| Português    | `pt`                 | `language: pt` or `language: pt-BR` |
| 繁體中文     | `zh-hk` 或者 `zh-tw` | `language: zh-hk`                   |
| Русский язык | `ru`                 | `language: ru`                      |
| Deutsch      | `de`                 | `language: de`                      |
| 日本語       | `ja`                 | `language: ja`                      |
| Indonesian   | `id`                 | `language: id`                      |
| Korean       | `ko`                 | `language: ko`                      |

想要切换语言只需要编辑**hexo 配置文件**`_config.yml`(注意区分主题配置文件)中的`language`项。
<!--more-->
## 设置菜单

对菜单外观的设置包括三个部分：

* 菜单的连接
* 菜单的名称
* 菜单的图标

### 设置菜单连接

NexT默认的菜单项有6个，初始只有`home`和`archives`，剩余菜单项需要在**主题配置文件**中手动开启。

| 键值       | 设定值                    | 显示文本（简体中文） |
| ---------- | ------------------------- | -------------------- |
| home       | `home: /`                 | 主页                 |
| archives   | `archives: /archives`     | 归档页               |
| categories | `categories: /categories` | ~~分类页~~           |
| tags       | `tags: /tags`             | ~~标签页~~           |
| about      | `about: /about`           | ~~关于页面~~         |
| commonweal | `commonweal: /404.html`   | ~~公益 404~~         |

例如我们想增加”标签页“这个菜单，需要以下3步：

1. 编辑**主题配置文件**中的`menu`项，该项初始内容如下:

   ```
   menu:
     home: / || home
     #about: /about/ || user
     #tags: /tags/ || tags
     #categories: /categories/ || th
     archives: /archives/ || archive
     #schedule: /schedule/ || calendar
     #sitemap: /sitemap.xml || sitemap
     #commonweal: /404/ || heartbeat
   ```

   将`tags`前的注释去掉。

2. 新建页面`tags`，既执行下列命令：
   `hexo new page "tags"`

3. 完成2后`blog/source/`中会出现`tags`目录，该目录下有`index.md`文件，文件内容如果：

   ```
   title: tags
   date: 2019-02-18 16:40:41
   ```

   编辑该文件，添加：`type: tags`。

完成以上3步后，重新生成静态文件，添加tags菜单项完成！

### 设置菜单名称

设置完tags后默认的中文名称是”标签“，那么如果想要改变菜单的显示名称怎么办？

Hexo 在生成菜单显示名称的时候是使用键值查找对应的语言翻译，并提取显示文本。这些翻译文本放置在 `next/languages/{language}.yml`（`{language}` 为你所使用的语言）。

以简体中文为例，若你需要添加一个菜单项，比如 `something`。那么就需要修改简体中文对应的翻译文件`languages/zh-Hans.yml`，在 `menu` 字段下添加一项：

```
menu:
  home: 首页
  archives: 归档
  categories: 分类
  tags: 标签
  about: 关于
  search: 搜索
  commonweal: 公益404
  something: 有料
```

### 设置菜单图标

NexT本身有默认的菜单图标，通过编辑**主题配置文件**中的`menu_icons`项来关闭或者开启。菜单图标的改变只能从“[Font Awesome 图标](https://fontawesome.com/)”中选择，如果想要改变默认的对应，只需：

```
menu_icons:
  enable: true
  #Icon Mapping.
  home: home #item name: icon name
  about: user
  categories: th
  tags: tags
  archives: archive
  commonweal: heartbeat
```



## 去除底部“*由 Hexo 强力驱动 | 主题 — NexT.Muse*“

博客底部样式在文件`next/layout/_partials/footer.swig`中定义，想要去除“*由 Hexo 强力驱动 | 主题 — NexT.Muse*“，只需在文件中删掉（或者注释掉）`{ % if theme.footer.powered %}`之后的所有内容。

![NexT_footer](https://my-blog-1256501598.cos.ap-beijing.myqcloud.com/github-page/learn/CS/Web/blog/NexT_footer.PNG)

## 设置背景动画

NexT自带5种背景动画，只需要在**主题配置文件**的相应位置开启即可。

## 其他

其他的一些外观设置可以参考Hexo配置文件和主题配置文件。

## 参考文档

1. [Hexo-NexT配置超炫网页效果](https://www.jianshu.com/p/9f0e90cc32c2)
2. [NexT文档](http://theme-next.iissnan.com/getting-started.html)
3. 可能漏掉某些参考文档，请作者联系添加引用或删除相关内容。
