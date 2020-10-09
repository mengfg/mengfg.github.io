
这是一个简洁的博客模板，响应式主题， 适配了电脑、手机各种屏幕，看效果直接点击下面链接

 * [Demo链接](https://mengfg.github.io/) （部署在github page）         

如果你喜欢请 Star ，你的 Star 是我持续更新的动力, 谢谢 😄.


### 环境要求

* Jekyll 支持: Mac 、Windows、ubuntu 、Linux 操作系统                     
* Jekyll 需要依赖: Ruby、bundler



### 目录结构

一个基本的 Jekyll 网站的目录结构一般是像这样的：

```
.
├── _config.yml
├── _includes
|   ├── footer.html
|   └── header.html
├── _layouts
|   ├── default.html
|   ├── post.html
|   └── page.html
├── _posts
|   └── 2016-10-08-welcome-to-jekyll.markdown
├── _sass
|   ├── _base.scss
|   ├── _layout.scss
|   └── _syntax-highlighting.scss
├── about.md
├── css
|   └── main.scss
├── feed.xml
└── index.html
```




### 提示

>* 如果你想使用我的模板，请把 _posts/ 目录下的文章都去掉。
>* 修改 _config.yml 文件里面的内容为你自己的个人信息。



### 博客主要模块介绍

> ### _config.yml
>
> ```
> _config.yml` 是博客的配置文件，整个站点的信息都在这修改，想要把我的模板改成你自己的也需要修改`_config.yml
> ```
>
> **重要字段说明**
>
> - enableToc: 是否开启文章自动生成目录，设置为false文章不会自动生成目录
> - comment/livere: livere评论系统，支持微信、qq、微博、豆瓣、twitter等登录后可以直接评论
> - comment/disqus: disqus评论系统，支持facebook、twitter等登录后可以直接评论
> - social/weibo、github、zhihu、jianshu等: 个人站底部展示的微博等三方社交按钮，点击后直接跳转到个人微博或其他社交主页
> - baidu/id: 百度统计，用来统计你个人站点的用户访问情况
> - ga/id: google统计，用来统计你个人站点的用户访问情况
>
> _config.yml 文件除以上字段还有一些可以自行修改，例如title之类的字段
>
> 
>
> ### _posts
>
> `_posts` 目录是用来存放文章的目录，写新文章，直接放在这个目录即可
>
> 
>
> ### 自定义页面
>
> about.md、support.md 等为自定义页面，如果你想添加自动以页面可以直接复制about.md 文件修改文件名和里面的内容即可。
>
> 如果需要在导航显示你新增的页面，直接在`_config.yml` 文件的nav字段中添加你新页面配置即可



### 编写文章

　　所有的文章都是 _posts 目录下面，文章格式为 mardown 格式，文章文件名可以是 .mardown 或者 .md。

　　编写一篇新文章很简单，你可以直接从 _posts/ 目录下复制一份出来 `2016-10-16-welcome-to-jekyll副本.markdown` ，修改名字为 2016-10-16-article1.markdown ，注意：文章名的格式前面必须为 2016-10-16- ，日期可以修改，但必须为 年-月-日- 格式，后面的 article1 是整个文章的连接 URL，如果文章名为中文，那么文章的连接URL就会变成这样的：http://leopardpan.cn/2015/08/%E6%90%AD%E5/ ， 所以建议文章名最好是英文的或者阿拉伯数字。 双击 2016-10-16-article1.markdown 打开

```
---
layout: post
title: layUI穿梭框 transfer
date: 2020-10-08
tags: layUI
---

正文...
```

title: 显示的文章名， 如：title: 我的第一篇文章
date: 显示的文章发布日期，如：date: 2016-10-16
tags标签的分类，如：tags: 随笔

注意：文章头部格式必须为上面的，…. 就是文章的正文内容。



### 效果预览

#### 头像效果

![](https://leopardpan.github.io/images/readme/icon.gif)

如果你只想要我博客里的头像效果，你只需要拿 leopardpan.github.io/_includes/side-panel.html 文件里面 `头像效果` 和 leopardpan.github.io/css/main.css 里面最后面 `头像效果` 部分就行了。

***

#### 博客首页   

![](https://leopardpan.github.io//images/readme/img4.png)   

***

#### 每篇文章下面都支持打赏   

![](https://leopardpan.github.io/images/readme/img3.png)

#### 文章详情   

![](https://leopardpan.github.io/images/readme/img1.png)


#### 文章支持标签分类 

![](https://leopardpan.github.io/images/readme/img2.png)

#### 手机端效果

<img width="300" src="https://leopardpan.github.io/images/readme/img5.png" alt="wechat">

#### 感谢   

本博客在https://leopardpan.github.io/基础上修改的。  