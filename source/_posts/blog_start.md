---
title:  "使用Jekyll在github上搭建博客"
date: 2016-05-03 00:00:00
categories: Jekyll
tags: Jekyll Github 搭建静态博客 blog
toc:
  on: true
  max_depth: 3
  nowrap: false
  list_number: true
toc_list_number: true
---


## 写在前面
>
>[Jekyll官方网站](http://jekyll.bootcss.com/)，内容很详细
>
>[作者阮一峰关于Jekyll与Github Blog的入门教程](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html)，推荐大家阅读
<!-- more -->

---



## 关于博客模板
网络上免费的Jekyll主题非常多，推荐2个主题网站

>
>[Jekyll主题网站一](http://jekyllthemes.org/)
>
>[Jekyll主题网站二](http://jekyllthemes.io/)

另外，本网站使用的主题来源

> [Gaohaoyang](http://gaohaoyang.github.io)

---

## 搭建过程中遇到的问题
由于Jekyll依赖于Ruby，在安装Ruby环境过程中由于众所周知的原因，总是遇到网络连接错误。
好在淘宝提供了RubyGems镜像服务，它与官方服务同步频率为15分钟。

> [淘宝网RubyGems镜像说明](https://ruby.taobao.org/)

以下为说明中提到的大致流程

> ```
> $ gem sources --add https://ruby.taobao.org/ --remove https://rubygems.org/
> $ gem sources -l
> *** CURRENT SOURCES ***
>
> https://ruby.taobao.org
> # 请确保只有 ruby.taobao.org
> $ gem install rails
> ```
