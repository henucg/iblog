---
layout: post
title:  "基于Jekyll和GithupPages搭建个人博客"
date:   2019-05-01 17:28:00 +0800
categories: 笔记
tags: Jekyll Githup 
excerpt: 
   "Jekyll 是一个简单的博客形态的静态站点生产机器。它有一个模版目录，其中包含原始文本格式的文档，通过 Markdown （或者 Textile）  以及 Liquid 转化成一个完整的可发布的静态网站。Jekyll也可以运行在GitHubPages上"
---


## 概述
+ Jekyll 是一个简单的博客形态的静态站点生产机器。它有一个模版目录，其中包含原始文本格式的文档，通过 Markdown （或者 Textile） 以及 Liquid 转化成一个完整的可发布的静态网站。Jekyll也可以运行在GitHubPages上

## 1、环境
+ MacOSMojave、Ruby2.3.7、Jekyll 3.8.5

## 2、基础准备
+ 本地安装Git相关服务
+ 申请Githup账号
+ Githup提交代码配置免密登陆

*详细过程此处不再赘述*

## 3、Githup新建博客仓库
+ 登陆Githup官网：[https://github.com/](https://github.com/){:target="_blank"}
+ 新建仓库iblog（名字可以自己修改），如图所示：
![]({{site.baseUrl}}/assets/20190501_01/0.png)
+ 新建完成之后将仓库clone到本地目录iblog/

## 4、本地安装Jekyll服务
+ 安装依赖环境  
   + 因为Jekyll是基于Ruby开发，所以开发前需要安装Ruby，Jekyll3.0之前，有一个语法高亮插件"Pygments"，它是基于python的，所以才会有各种教程里面都说搭建jekyll之前需要python环境,但是,jekyll3.0以后，语法高亮插件已经默认改成了“rouge‘而它是基于ruby的，也就是说现在搭建jekyll,我们完全不必要再安装python  
   检查本机是否安装了Ruby：  
   `ruby -v`
+ 安装Jekyll和bundler gems  
   `gem install jekyll bundler`
+ 执行完成检查是否成功  
   `jekyll -v` 
+ 详细参考Jekyll官方文档：[https://www.jekyll.com.cn/](https://www.jekyll.com.cn/){:target="blank"}

## 5、本地搭建博客
+ 下载模版
   + 在Jekyll官方主题网站上下载自己喜欢的主题模版：[http://jekyllthemes.org/](http://jekyllthemes.org/){:target="_blank"} 
   + 或者在Githup上引用别人的模版
+ 修改与调试
   + 解压模版压缩包，将文件里面的内容拷贝到自己的项目目录iblog中，如图：
   ![]({{site.url}}/assets/20190501_01/1.png){:height="300" width="400"}
      + 其中_layouts目录是模版框架，需要修改样式可以根据自己需要修改里面的文件
      + _posts目录下面放的是博客文件，我用的是markDown格式文件，文件名称：  
      	YYYY-MM-DD-博客名称.md  
      	Jekyll会自动根据名称解析博客
      + 新建博客只需按照名称规则新建文件放到_posts目录下即可
   + 命令行进入目录/iblog/
   + 编译：
   `jekyll build -v`
   + 启动：
   `jekyll server`
   + 如果成功显示：
   ![]({{site.baseUrl}}/assets/20190501_01/2.png){:height="300" width="600"}
+ 本地访问
   + 在浏览器中访问：http://127.0.0.1:4000，即可以看到自己博客

## 6、提交与发布
+ 将代码提交到Githup
+ 配置GithupPages
   + 登陆Githup，找到iblog仓库，点击Settings
   + 选择需要发布在GithupPages上的分支，如果只有一个分支就是master：
   ![]({{site.baseUrl}}/assets/20190501_01/3.png)
+ 访问：[https://henucg.github.io/iblog/](https://henucg.github.io/iblog/){:target="_blank"}，路径根据自己的githup名称修改
+ 配置域名
   + 在iblog/目录下添加文件CNAME，在文件里面写上自己的域名即可，例如：www.iblog.com

## 7、注意点
+ _posts文件夹下文件名称：YYYY-MM-DD-博客名称.md
   + 如果文件名称中的日期超过当前系统时间，则Jekyll不能识别该文件，因此文件中的日期不能大于当前系统日期
+ 静态文件路径问题
   + 本机调试的时候根路径与发布到githup之后根路径可能不一致，导致在本机静态文件可以正常加载，而在githup上无法加载，需要根据实际情况修改路径