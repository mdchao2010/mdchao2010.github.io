---
layout: post
title:  "Jekyll 搭建静态博客"
date:   2019-01-30 
categories: jekyll
tags: jekyll RubyGems
---

* content
{:toc}

博客的搭建教程修改自[gaohaoming](http://gaohaoyang.github.io/)



## 概述

>`Jekyll` 基于Ruby的静态网页生成系统，采用模板将Markdown(或Textile)文件转换为统一的网页
>`GitHub Pages` 免费的静态站点，三个特点：免费托管、自带主题、支持自制页面和Jekyll

## 搭建过程

要求：拥有个人的github账号

首先，在你的github上建立一个以xxx.github.io为命名的代码仓库，其中xxx代表的是你的github账号名，如我的账号名是[mdchao2010](http://mdchao2010.github.io/)，则建立的是
mdchao2010.github.io，同时在底部Add .gitigore选择Jekyll模板，这样Jekyll产生的临时文件，当然你也可以以后手动配置

建立完代码仓库后，将代码仓库克隆到本地
```bash
$ git clone https://github.com/xxx/xxx.github.io.git
```
然后在本地的代码仓库里面创建一个测试页面并推送到github的代码仓库
```bash
$ cd xxx.github.io
$ echo "Hello World" > index.html
$ git add --all
$ git commit -m "Initial commit"
$ git push -u origin master
```
推送完代码后等一段时间，快的话几分钟，慢的话要二十分钟或者半个小时吧，等github运行编译你的代码，
完了之后在浏览器输入你的项目名称：xxx.github.io，如果一切正常，你就能看到一个显示Hello World的页面

### 安装配置jekyll

Windows 用户可以直接使用 [RubyInstaller](http://rubyinstaller.org/)  安装 ruby 环境。后续的操作中可能还会提示安装 DevKit，根据提示操作即可。


### 安装RubyGems

官网下载 [https://rubygems.org/pages/download](http://rubygems.org/pages/download) rubygems-3.0.2.zip,解压到你想要的目录下，路径不要有空格，
然后cmd命令指到这个文件夹下面，输入`ruby setup.rb`执行安装，

### 用RubyGems安装Jekyll

执行下面的语句安装   
```bash
gem install jekyll
gem install jekyll-paginate
```

至此jekyll就已经安装完毕了，后续就是个性化的自己设定了。

### 创建博客

cd到github克隆的工作目录   
`jekyll serve` 来启动本地服务器，默认使用4000端口，如果需要更换端口，可以运行`jekyll serve -P $POST` 指定其他的端口。
添加 `--watch` 参数检测文件夹内的变化，即修改后不需要重新启动jekyll
在浏览器输入：`127.0.0.1:4000` 就可以进行本地预览了，建议使用webstorm开发

