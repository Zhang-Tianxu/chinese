---
title: github pages + hexo + next + Travis CI实现免费个人博客
date: 2021-03-08 14:55:09
tags:
    - GitHub Pages
categories:
    - 自媒体
    - 博客
---
[GitHub Pages](https://pages.github.com/)是GitHub提供的静态网页托管工具，可以用来建立个人网页，也可以建立网页介绍某个项目。

[Hexo](https://hexo.io/zh-cn/)是一个博客框架，而[NexT](https://theme-next.iissnan.com/)是Hexo的一个主题。

最后Travis CI是针对GitHub的一款持续集成工具，这里用来完成网站的自动化部署。
<!--more-->
# GitHub Pages设置
GitHub Pages主页中有建站的简单教程，为了支持MarkDown撰写博客，选择使用博客框架Hexo，并选择一个比较流行的框架NexT。

在此基础上可以添加许多功能：
1. 置顶
    1. 移除默认安装的插件`npm uninstall hexo-generator-index --save`
    2. 安装新插件`npm install hexo-generator-index-pin-top --save`
    3. 在需要置顶的文章头部`top: true`或`top:整数`，其中整数越大的文章越靠前
    4. 为置顶的文章添加置顶标签,在`/themes/next/layout/_macro/post.swig`文件的`<div class="post-meta">`下方，插入如下代码：
    ```
        {% if post.top %}
        <i class="fa fa-thumb-tack"></i>
        <font color=7D26CD>置顶</font>
        <span class="post-meta-divider">|</span>
        {% endif %}
    ```
2. 数学公式
    1. 编辑`theme/next/_config.yml`
    ```
    # MathJax Support
    mathjax:
    enable:  true
    per_page: false
    cdn: //cdn.bootcss.com/mathjax/2.7.1/latest.js?config=TeX-AMS-MML_HTMLorMML
    ```
    2. 为了更好的性能，不选择在所有页面下支持数学公式。在需要支持matchjax的文章头部，添加`mathjax: true`
3. 评论功能
    1. 评论功能和阅读统计都可以使用LeanCloud
    2. 编辑`theme/next/_config.yml`
        ```
            valine:
                enable: true
                appid:  xxxxxxxx
                appkey:  yyyyyyyyyyy
                notify: false # mail notifier , https://github.com/xCss/Valine/wiki
                verify: false # Verification code
                placeholder: 评论 # comment box placeholder
                avatar: mm # gravatar style
                guest_info: nick,mail,link # custom comment header
                pageSize: 10 # pagination size
        ```
{% img commentsImg https://tva1.sinaimg.cn/large/008eGmZEgy1gocmlo8c9sj317k0o8myh.jpg 400 600 '"评论功能" "加载失败"' %}
4. 阅读统计
    [为NexT主题添加文章阅读量统计功能](https://notes.doublemine.me/2015-10-21-%E4%B8%BANexT%E4%B8%BB%E9%A2%98%E6%B7%BB%E5%8A%A0%E6%96%87%E7%AB%A0%E9%98%85%E8%AF%BB%E9%87%8F%E7%BB%9F%E8%AE%A1%E5%8A%9F%E8%83%BD.html#%E9%85%8D%E7%BD%AELeanCloud)
5. 字数统计
    1. 编辑`theme/next/_config.yml`
        ```
            post_wordcount:
            item_text: true
            wordcount: true
            min2read: true
            totalcount: true
            separated_meta: true
        ```
    2. 执行`npm install hexo-wordcount@2 --save`，安装需要的库
{% img wordCountImg https://tva1.sinaimg.cn/large/008eGmZEgy1gocmhvhwsnj30cw01iwei.jpg 200 30 '"字数统计" "加载失败"' %}
6. 添加网易云播放器
    1. 去网易云音乐找一首喜欢的歌。
    2. 点击“生成外链播放器”，复制HTML代码。
    3. 将HTML代码添加到`/themes/hexo-theme-next/layout/_macro/sidebar.swig`中`<aside id="sidebar" class="sidebar”>`后面，并用`<div>`包裹。
{% img musicImg https://tva1.sinaimg.cn/large/008eGmZEgy1gocm3k2n6uj30ia0lujua.jpg 400 200 '"网易云音乐" "加载失败"' %}
7. 将标签云改为彩色
    1. 在`themes/next/layout/`中新建`tag-color.swig`文件，代码为：
        ```
<script type="text/javascript">
     var alltags = document.getElementsByClassName('tag-cloud-tags');
     var tags = alltags[0].getElementsByTagName('a');
     for (var i = tags.length - 1; i >= 0; i--) {
       var r=Math.floor(Math.random()*75+130);
       var g=Math.floor(Math.random()*75+100);
       var b=Math.floor(Math.random()*75+80);
       tags[i].style.background = "rgb("+r+","+g+","+b+")";
     }
</script>

<style>
  .tag-cloud-tags{
    /*font-family: Helvetica, Tahoma, Arial;*/
    /*font-weight: 100;*/
    text-align: center;
    counter-reset: tags;
  }
  .tag-cloud-tags a{
    border-radius: 6px;
    padding-right: 5px;
    padding-left: 5px;
    margin: 8px 5px 0px 0px;
  }
  .tag-cloud-tags a:before{
    content: "?";
  }

  .tag-cloud-tags a:hover{
     box-shadow: 0px 5px 15px 0px rgba(0,0,0,.4);
     transform: scale(1.1);
     /*box-shadow: 10px 10px 15px 2px rgba(0,0,0,.12), 0 0 6px 0 rgba(104, 104, 105, 0.1);*/
     transition-duration: 0.15s;
  }
</style>
        ```
    2. 在`/themes/next/layout/page.swig`中引入`tag-color.swig`，即在`<div class="tag-cloud">`代码段下方添加`{ % include 'tag-color.swig' % }`
    3. 也可以将标签云直接加入主页，在`/themes/next/layout/index.swig`中的block content代码块中加入以下代码：
```
<div class="tag-cloud">
	  <div class="tag-cloud-tags" id="tags">
		{{ tagcloud({min_font: 16, max_font: 16, amount: 300, color: true, start_color: '#fff', end_color: '#fff'}) }}
	  </div>
	</div>
	<br>
	
	{% include 'tag-color.swig' %}
```

8. 展示近期文章
    1. 修改`themes/next/layout/_macro/sidebar.swig` 。找到`theme.social`，在该板块后隔一行添加如下代码。
    ```
    {# recent posts #}
    {% if theme.recent_posts %}
    <div class="links-of-blogroll motion-element {{ "links-of-blogroll-" + theme.recent_posts_layout  }}">
        <div class="links-of-blogroll-title">
            <!-- modify icon to fire by szw -->
            <i class="fa fa-history fa-{{ theme.recent_posts_icon | lower }}" aria-hidden="true"></i>
            {{ theme.recent_posts_title }}
        </div>
        <ul class="links-of-blogroll-list">
            {% set posts = site.posts.sort('-date') %}
            {% for post in posts.slice('0', '5') %}
            <li class="recent_posts_li">
                <a href="{{ url_for(post.path) }}" title="{{ post.title }}" target="_blank">{{ post.title }}</a>
             </li>
             {% endfor %}
        </ul>
    </div>
    {% endif %}
    ```
    2. 编辑`themes/next/source/css/_common/components/sidebar/sidebar-blogroll.styl`
```
li.recent_posts_li {
    text-align: cengter;
    display: block;
    word-break: keep-all;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}
```

    3. 在`themes/next/_config.yml`中添加下方代码
```
# 近期文章设置
recent_posts_title: 近期文章
recent_posts_layout: block
recent_posts: true
```
{% img rencentPostImg https://tva1.sinaimg.cn/large/008eGmZEgy1gocmksvxlvj30da0460su.jpg 200 100 '"近期文章" "加载失败"' %}

# Travis CI自动部署GitHub Pages
[Travis官方教程](https://docs.travis-ci.com/user/deployment/pages/)
有了Travis CI，更换电脑时，不需要在本地配置完整的环境，可以直接修改md文件，push到github后，Travis CI会自动生成和部署，非常的方便。
```
sudo: false
language: node_js
node_js:
  - 10 # use nodejs v10 LTS
before_script: # 配置环境
    - npm install hexo-generator-searchdb --save # 用于支持本地搜索功能
    - npm uninstall hexo-generator-index --save 
    - npm install hexo-generator-index-pin-top --save # 这两行用于支持置顶功能
    - npm install --save hexo-filter-flowchart # 用于支持markdown中的流程图功能
    - npm install hexo-wordcount@2 --save #用于支持字数统计功能
cache: npm
branches:
  only:
    - master # build master branch only
script:
  - hexo generate # generate static files
deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GH_TOKEN
  keep-history: true
  on:
    branch: master
  local-dir: public


notifications:
   email:
     recipients:
       - xxx@xxx.com
         #-
     on_success: never # default: change
     #on_success: change # default: change
     on_failure: always # default: always
```
