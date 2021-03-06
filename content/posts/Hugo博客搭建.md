---
title: "Hugo博客搭建"
date: 2020-08-21T11:05:18+08:00
author: 小叽
categories: ["blog"]
tags: ["blog"]
draft: false
---

Hugo 是一个由 Go 语言实现的静态页面生成器，是世界上最快的博客生成器。

<!--more-->

# Hugo简介
Hugo 是一个由 Go 语言实现的静态页面生成器，所有页面都是通过 Markdown (标记语言)开发。
它能够快速的在GitHub上搭建个人静态博客。它使用起来非常简单，相对于Jekyll复杂的安装设置来说，
Hugo 仅需要一个二进制文件 hugo(hugo.exe) 即可轻松用于本地调试和生成静态页面。  
Hugo 是一种通用的网站框架。严格来说，Hugo 应该被称作静态网站生成器。  
静态网站生成器从字面上来理解，就是将你的内容生成静态网站。所谓“静态”的含义其实反映在网站页面的生成的时间。一般的web服务器（WordPress, Ghost, Drupal等等）在收到页面请求时，需要调用数据库生成页面（也就是HTML代码），再返回给用户请求。而静态网站则不需要在收到请求后生成页面，而是在整个网站建立起之前就将所有的页面全部生成完成，页面一经生成便称为静态文件，访问时直接返回现成的静态页面，不需要数据库的参与。

Hugo官网：[https://gohugo.io/](https://gohugo.io/)

# 安装
## Mac安装方式
````
brew install hugo
hugo version
````

## windows 安装方式
1. 在 [Hugo Releases](https://github.com/gohugoio/hugo/releases) 页面上下载hugo_XX_Windows-64bit.zip。
2. 下载的压缩包解压到 hugo 目录 例如：D:/Software/hugo 。
3. 将压缩包的解压路径添加到环境变量PATH中。
4. 重启机器，在cmd窗口输入  `hugo version` 查看Hugo是否安装成功。

# 创建本地博客
## 生成站点
````
hugo new site quickstart
````
其中quickstart为本地博客站点的名称，理论上是随意的，但是推荐为 XXX_github.io-creator

## 添加主题
Hugo官方主题：[https://themes.gohugo.io/](https://themes.gohugo.io/)
这里使用git克隆方式下载主题。
````
cd quickstart(刚刚新建站点的目录)
git init
git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke
````
然后，将这些主题添加到站点的配置文件中(config.toml)
````
echo 'theme = "ananke"' >> config.toml
````

## 添加博客
其中my-first-post为博客名，可以随意更改
````
hugo new posts/my-first-post.md
````
打开新建的md(MarkDown)文件，文件中的内容显示为
````
title: "My First Post"
date: 2019-03-26T08:47:11+01:00
draft: true
````
注意：默认创建的是草稿类型，需要将draft值改为false才能看到页面。

## 运行Hugo Server
````
hugo server -D
````
紧接着你会看到
````
                  | EN
+------------------+----+
  Pages            | 10
  Paginator pages  |  0
  Non-page files   |  0
  Static files     |  3
  Processed images |  0
  Aliases          |  1
  Sitemaps         |  1
  Cleaned          |  0

Total in 11 ms
Watching for changes in /Users/bep/quickstart/{content,data,layouts,static,themes}
Watching for config changes in /Users/bep/quickstart/config.toml
Environment: "development"
Serving pages from memory
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
````
在你的浏览器里面输入：localhost:1313
你在md文件中添加的内容就会展示到浏览器中，现在一个本地博客站点就搭建完成了。所有静态页面都会生成到 public 目录下，生成静态网站后并push到你的GitHub Pages上，就能得到一个在线的个人博客了。

# 上传(发布)Git
在Git新建仓库XXX_github.io，将生成的public文件夹上传到仓库XXX_github.io项目中，并提交版本。
现在你就可以用浏览器输入：[jmh1234.github.io](jmh1234.github.io) 就可以访问博客了。

## GitHub Pages
查看[GitHub Pages](https://guides.github.com/features/pages/)在线帮助文档
