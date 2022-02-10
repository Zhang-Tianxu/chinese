---
title: 微信小程序：scroll-view与swiper
tags:
  - 微信
  - 微信小程序
  - 小程序
  - 组件
categories:
  - 专业学习
  - 前端
  - 小程序
  - 微信小程序
date: 2020-04-08 11:51:08
---

# scroll-view与swiper的区别
在做小程序的时候有一个需求:主要内容做成卡片，然后通过划动在卡片之间切换。划动的内容，第一反应就想到scroll-view组件，没细想就开始做了，但是做的差不多了总出现一些小毛病：
1. 划动时会停在两个卡片中间，而不是想要的一个卡片一个卡片的划动。
2. scroll-view带惯性感性，大力划动会连续跳过好几个卡片。还是没达到一个卡片一个卡片的划动。

为了解决这个问题，我尝试了很多方法。第1个问题，我通过用js设置scroll-view的划动距离，强行让划到中间的卡片复位。废了九牛二虎之力才解决了，第2个问题怎么也解决不了。最后才发现**更符合我需求的组件是swiper！**

<!--more-->

参考：
* [scroll-view](https://developers.weixin.qq.com/miniprogram/dev/component/scroll-view.html)
    * 滚动视图容器
* [swiper](https://developers.weixin.qq.com/miniprogram/dev/component/swiper.html)
    * 滑块视图容器


