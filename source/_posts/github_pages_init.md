---
title:  "基于Github搭建个人博客"
date: 2016-10-20 00:00:00
categories: Github Jekyll Hexo
tags: Jekyll Hexo blog Github
toc:
  on: true
  max_depth: 3
  nowrap: false
  list_number: true
toc_list_number: true
---

# 基于Github搭建个人博客

## Github Pages

### 什么是github pages？

GitHub Pages is a static site hosting service.

GitHub Pages is designed to host your personal, organization, or project pages directly from a GitHub repository. To learn more about the different types of GitHub Pages sites, see "[User, organization, and project pages](https://help.github.com/articles/user-organization-and-project-pages/)."

You can create and publish GitHub Pages online using the [Automatic Page Generator](https://help.github.com/articles/creating-pages-with-the-automatic-generator). If you prefer to work locally, you can use [GitHub Desktop](http://desktop.github.com) or the [command line](https://help.github.com/articles/adding-an-existing-project-to-github-using-the-command-line).



 由用户编写的、免费的、托管在github上的静态网页

---

<!-- more -->

### 如何搭建Github Pages 

官方教程

 https://pages.github.com/

https://help.github.com/articles/user-organization-and-project-pages/

搭建github博客基本分为如下几步：

* 第一步，create a repository
* 第二步， clone the repository
* 第三步，create your website pages
* 第四步，commit and push your files



github pages为我们搭建的个人博客地址是有规则的：
例如，我的github地址为https://github.com/wr4javaee
那么，我的博客主页地址为https://wr4javaee.github.io

若我的项目名称叫blog，那么对应的pages地址就是https://wr4javaee.github.io/blog



* 注册github账号

https://github.com/join

onepieceworld@qq.com&3377128016，protosssoul@qq.com&2027194199

![](/images/blog_create/create_repository.png)

github为我们准备了两种搭建博客方式，一种是github提供模板，我们只需写md文档即可。另一种是从头搭建。

我们先介绍第一种方式。



#### 方法一，利用github pages模板搭建博客

进入项目->settings->github pages->launch

![](/images/blog_create/settings_lunch.png)



点击launch后，进入

![](/images/blog_create/lunch_page1.png)





填写好项目名称、项目描述后，进入下一步，选择项目模板

![](/images/blog_create/settings_theme.png)

选择好模板后，点击 publish page即可提交。

稍等片刻后，github即可为我们搭建好项目对应的网站。

通过访问https://onepieceworld.github.io/test/




#### 方法二，从头搭建博客

若我们查看github模板为我们创建的博客，会发现github为我们自动创建了一个分支gh-pages作为默认分支，博客的文件都提交在这个特定的分支中。

![](/images/blog_create/git_branch.png)

那么，若我们从头搭建博客，就得模仿这一步骤。github规定，只有该分支中的页面，才会生成静态网页文件。

我们新创建一个reponsitory，名为test1

https://github.com/onepieceworld/test1.git



接下来，我们创建一个全新的gh-pages分支，并将其作为默认分支

```
yumao@ubuntu:~/github/test1$ git checkout --orphan gh-pages
```



为新的空白分支创建默认页面并提交

```
yumao@ubuntu:~/github/test1$ git checkout --orphan gh-pages
Switched to a new branch 'gh-pages'
yumao@ubuntu:~/github/test1$ git branch 
yumao@ubuntu:~/github/test1$ echo "Hello World!" > index.html
yumao@ubuntu:~/github/test1$ git add index.html 
yumao@ubuntu:~/github/test1$ git commit -a -m "commit index pages"
[gh-pages (root-commit) 0fab7c5] commit index pages
 1 file changed, 1 insertion(+)
 create mode 100644 index.html
yumao@ubuntu:~/github/test1$ git push origin gh-pages 
```



提交成功后，访问https://onepieceworld.github.io/test1/，

可以看到，github已经自动为我们将静态文件部署。

---



## 利用Jekyll搭建个人博客

[Jekyll官方网站](http://jekyll.com.cn)
[Jekyll github](https://github.com/jekyll/jekyll)


### 什么是Jekyll？

Jekyll is a simple, blog-aware, static site generator perfect for 
personal, project, or organization sites. Think of it like a file-based 
CMS, without all the complexity. Jekyll takes your content, renders 
Markdown and Liquid templates, and spits out a complete, static website 
ready to be served by Apache, Nginx or another web server. Jekyll is the
 engine behind [GitHub Pages](https://pages.github.com), which you can use to host sites right from your GitHub repositories.

一个可以将纯文本转化为静态网站文件的生成器。

基于Jekyll，我们可以只关注编写纯文本，如markdown，而无需将精力放在HTML本身。



### Jekyll的简易用法

> [Jekyll官方使用简介](http://jekyll.bootcss.com/)，内容很详细
> [作者阮一峰关于Jekyll与Github Blog的入门教程](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html)，推荐大家阅读



首先，Jekyll依赖Ruby

安装Ruby环境过程中会遇到GWF，需要用到淘宝提供的RubyGems镜像服务，它与官方服务同步频率为15分钟。

[淘宝RubyGems镜像]: https://ruby.taobao.org/



Ruby环境搭建好后，我们通过以下命令安装Jekyll

```
~ $ gem install jekyll
```



接下来，利用jekyll命令jekyll new my-awesome-site自动创建jekyll项目

```
yumao@ubuntu:~/github$ jekyll new jekyll_test
New jekyll site installed in /home/yumao/github/jekyll_test. 
Running bundle install in /home/yumao/github/jekyll_test... 
Fetching gem metadata from https://rubygems.org/................
Fetching version metadata from https://rubygems.org/...
Fetching dependency metadata from https://rubygems.org/..
Resolving dependencies...
Using addressable 2.4.0
Using colorator 1.1.0
Using ffi 1.9.14
Using forwardable-extended 2.6.0
Using sass 3.4.22
Using rb-fsevent 0.9.7
Using kramdown 1.12.0
Using liquid 3.0.6
Using mercenary 0.3.6
Using rouge 1.11.1
Using safe_yaml 1.0.4
Using minima 2.0.0
Using bundler 1.13.5
Using rb-inotify 0.9.7
Using pathutil 0.14.0
Using jekyll-sass-converter 1.4.0
Using listen 3.0.8
Using jekyll-watch 1.5.0
Using jekyll 3.3.0
Using jekyll-feed 0.8.0
Bundle complete! 3 Gemfile dependencies, 20 gems now installed.
Use `bundle show [gemname]` to see where a bundled gem is installed.
yumao@ubuntu:~/github$ 
```



进入项目目录下，执行Jekyll命令 jekyll serve，启动项目

```
yumao@ubuntu:~/github$ cd jekyll_test/
yumao@ubuntu:~/github/jekyll_test$ jekyll serve
WARN: Unresolved specs during Gem::Specification.reset:
      rouge (~> 1.7)
      jekyll-watch (~> 1.1)
WARN: Clearing out unresolved specs.
Please report a bug if this causes problems.
Configuration file: /home/yumao/github/jekyll_test/_config.yml
Configuration file: /home/yumao/github/jekyll_test/_config.yml
            Source: /home/yumao/github/jekyll_test
       Destination: /home/yumao/github/jekyll_test/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
                    done in 0.286 seconds.
 Auto-regeneration: enabled for '/home/yumao/github/jekyll_test'
Configuration file: /home/yumao/github/jekyll_test/_config.yml
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.

```

![](/images/blog_create/jekyll_default_page.png)



至此，一个简易的jekyll项目就搭建好了，我们将它部署到github中，每次只要新增md文档，提交到github后，其会自动编译jekyll项目，自动解析成静态网页文件。



### Jekyll的目录结构




![](/images/blog_create/jekyll_files.png)

```
.
├── _config.yml
├── _drafts
|   ├── begin-with-the-crazy-ideas.textile
|   └── on-simplicity-in-technology.markdown
├── _includes
|   ├── footer.html
|   └── header.html
├── _layouts
|   ├── default.html
|   └── post.html
├── _posts
|   ├── 2007-10-29-why-every-programmer-should-play-nethack.textile
|   └── 2009-04-26-barcamp-boston-4-roundup.textile
├── _data
|   └── members.yml
├── _site
└── index.html
```



| 文件 / 目录                                  | 描述                                       |
| ---------------------------------------- | ---------------------------------------- |
| `_config.yml`                            | 保存[配置](http://jekyll.com.cn/docs/configuration/)数据。很多配置选项都会直接从命令行中进行设置，但是如果你把那些配置写在这儿，你就不用非要去记住那些命令了。 |
| `_drafts`                                | drafts 是未发布的文章。这些文件的格式中都没有 `title.MARKUP` 数据。学习如何使用 [drafts](http://jekyll.com.cn/docs/drafts/). |
| `_includes`                              | 你可以加载这些包含部分到你的布局或者文章中以方便重用。可以用这个标签          `{nclude file.ext`          来把文件 `_includes/file.ext` 包含进来。 |
| `_layouts`                               | layouts 是包裹在文章外部的模板。布局可以在 [YAML 头信息](http://jekyll.com.cn/docs/frontmatter/)中根据不同文章进行选择。          这将在下一个部分进行介绍。标签          `content`          可以将content插入页面中。 |
| `_posts`                                 | 这里放的就是你的文章了。文件格式很重要，必须要符合:          `YEAR-MONTH-DAY-title.MARKUP`。          The [permalinks](http://jekyll.com.cn/docs/permalinks/) 可以在文章中自己定制，但是数据和标记语言都是根据文件名来确定的。 |
| `_data`                                  | Well-formatted site data should be placed here. The jekyll engine will           autoload all yaml files (ends with `.yml` or `.yaml`)           in this directory. If there's a file `members.yml` under the directory,           then you can access contents of the file through `site.data.members`. |
| `_site`                                  | 一旦 Jekyll 完成转换，就会将生成的页面放在这里（默认）。最好将这个目录放进你的 `.gitignore` 文件中。 |
| `index.html` and other HTML, Markdown, Textile files | 如果这些文件中包含 [YAML 头信息](http://jekyll.com.cn/docs/frontmatter/) 部分，Jekyll 就会自动将它们进行转换。当然，其他的如 `.html`， `.markdown`，          `.md`，或者 `.textile` 等在你的站点根目录下或者不是以上提到的目录中的文件也会被转换。 |
| Other Files/Folders                      | 其他一些未被提及的目录和文件如          `css` 还有 `images` 文件夹，          `favicon.ico` 等文件都将被完全拷贝到生成的 site 中。 这里有一些[使用 Jekyll 的站点](http://jekyll.com.cn/docs/sites/)，如果你感兴趣就来看看吧。 |



#### _config.yml配置文件

[官方配置文件说明](http://jekyll.com.cn/docs/configuration/)



### 使用Jekyll的开源模板

网络上免费的Jekyll主题非常多，推荐2个主题网站

>
>[Jekyll主题网站一](http://jekyllthemes.org/)
>
>[Jekyll主题网站二](http://jekyllthemes.io/)



这里为大家推荐一个我喜欢的主题来源

> [Gaohaoyang](http://gaohaoyang.github.io)

github地址

https://github.com/Gaohaoyang/gaohaoyang.github.io.git



### Jekyll的优缺点

Jekyll是比较流行的博客系统，其主题、插件数量上比较丰富，且上手简单。
缺点为文章量多的时候，生成速度慢。

---



## 利用Hexo搭建个人博客

### Hexo与Jekyll的区别
Hexo是基于Node.js编写的静态博客系统，使用Jekyl我们可以将原生的markdown文档上传到github，由github自动解析成静态html文件。但hexo为本地将markdown文档解析后，将生成的静态html文件直接上传到github中。

由于hexo基于nodejs，因此其生成html速度较Jekyll快，相对更灵活。主题数量然有Jade/Stylus/Less等各种方言支持。

|        | 语言     | 活跃度  | 开箱即用 | 主题数量 | 主题系统 | 生成速度 | 博客适应性 | 非博客适应性 |
| ------ | ------ | ---- | ---- | ---- | ---- | ---- | ----- | ------ |
| Jekyll | Ruby   | S    | B    | A    | A    | C    | A     | C      |
| Hexo   | NodeJS | B    | A    | A    | B    | A    | A     | B      |



### Hexo官方文档

[Hexo官方文档](https://hexo.io/zh-cn/docs/)



### 安装Hexo

Hexo基于Node.js，因此首先应安装Node.js环境。

cURL：

```
$ curl https://raw.github.com/creationix/nvm/master/install.sh | sh
```

Wget
```
$ wget -qO- https://raw.github.com/creationix/nvm/master/install.sh | sh
```
安装完成后，重启终端执行下列命令完成安装Node.js
```
$ nvm install stable
```

若环境准备就绪，使用以下命令安装Hexo

```
$ npm install -g hexo-cli
```



### 使用Hexo建立网站

使用以下命令初始化网站

```
$ hexo init <folder>
$ cd <folder>
$ npm install
```



建站后的文件目录如下：

![](/images/blog_create/hexo_files.png)



```
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```

* scaffolds模板文件夹

建站的模板，hexo依赖于此模板来生成静态文件。

*  _config.yml配置文件

[hexo官方配置文件说明](https://hexo.io/zh-cn/docs/configuration.html)

以下为hexo的默认配置

```
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
# 网站标题
title: Hexo 
# 网站副标题
subtitle: 
# 网站描述
description: 
author: John Doe
# 网站使用的语言
language: 
# 网站时区。Hexo 默认使用您电脑的时区。时区列表。比如说：America/New_York, Japan, 和 UTC 。
timezone: 

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
# 如果您的网站存放在子目录中，例如 http://yoursite.com/blog，则请将您的 url 设为 #http://yoursite.com/blog 并把 root 设为 /blog/。
url: http://yoursite.com
# 网站根目录
root: /
# 文章的 永久链接 格式 	:year/:month/:day/:title/
permalink: :year/:month/:day/:title/
# 永久链接中各部分的默认值 	
permalink_defaults:

# Directory
# 资源文件夹，这个文件夹用来存放内容。
source_dir: source
# 公共文件夹，这个文件夹用于存放生成的站点文件
public_dir: public
# 标签文件夹
tag_dir: tags
# 归档文件夹
archive_dir: archives
# 分类文件夹
category_dir: categories
# Include code 文件夹
code_dir: downloads/code
# 国际化（i18n）文件夹
i18n_dir: :lang
# 跳过指定文件的渲染，您可使用 glob 表达式来匹配路径。 	
skip_render:

# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace:

# Category & Tag
# 默认分类 	uncategorized
default_category: uncategorized
# 分类别名 	
category_map:
# 标签别名 	
tag_map:

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
# 日期格式 	YYYY-MM-DD
date_format: YYYY-MM-DD
# 时间格式 	H:mm:ss
time_format: HH:mm:ss

# Pagination
## Set per_page to 0 to disable pagination
# 每页显示的文章量 (0 = 关闭分页功能) 	10
per_page: 10
# 分页目录 	page
pagination_dir: page

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
# 当前主题名称。值为false时禁用主题
theme: landscape

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type:
```



* source资源文件夹

用来存放用户资源，其中_post文件夹下的markdown文档，会被hexo自动解析成html，并放置到public文件夹中

其他以_开头的文件夹会被自动忽略

* themes主题文件夹

hexo在解析时，会根据主题来生成静态页面。



#### 新建一篇文章

```
$ hexo new [layout] <title>
```

layout若不指定，默认从_config.yml文件中的default_layout代替



#### 生成静态文件

```
$ hexo generate
```

| 选项               | 描述          |
| ---------------- | ----------- |
| `-d`, `--deploy` | 文件生成后立即部署网站 |
| `-w`, `--watch`  | 监视文件变动      |



使用generate命令后，hexo的文件夹会自动生成静态文件，放置到配置文件中规定的public目录中

![](/images/blog_create/hexo_files_after_generate_1.png)



public文件夹中的内容如下

![](/images/blog_create/hexo_files_after_generate_2.png)



#### 启动服务器

```
$ hexo server
```



#### 部署网站

```
$ hexo deploy
```

| 参数                 | 描述           |
| ------------------ | ------------ |
| `-g`, `--generate` | 部署之前预先生成静态文件 |



部署后的界面如下
![](/images/blog_create/hexo_server_page.png)



#### 清除缓存文件

```
$ hexo clean
```

清除缓存文件 (`db.json`) 和已生成的静态文件 (`public`)。

![](/images/blog_create/hexo_clean.png)



#### 其他hexo命令

[其他hexo命令](https://hexo.io/zh-cn/docs/commands.html)



### hexo部署到github中

通过_config.yml文件建立关联，

```
deploy:
  type: git
  repository: https://github.com/onepieceworld/hexo_test.git
  branch: gh-pages
```

执行命令

```
~ $ npm install hexo-deployer-git --save
~ $ hexo deploy
```



至此，hexo已成功将解析后的html文件上传到github中

![](/images/blog_create/hexo_github.png)

