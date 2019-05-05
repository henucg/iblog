---
layout: post
title:  "JS中的location属性介绍"
date:   2019-05-02 17:39:00 +0800
categories: 笔记
tags: JS
excerpt: "在基于Jekyll和GithupPages搭建个人博客这篇笔记当中，使用到了JS的Location的一些属性，在此总结一下"
---


## 1、概述
在[基于Jekyll和GithupPages搭建个人博客](/笔记/2019/05/01/基于Jekyll和GithupPages搭建个人博客.html){:target="blank"}这篇笔记当中，使用到了JS的Location的一些属性，在此总结一下

## 2、location对象基本介绍
location是最有用的DOM对象之一，它提供了与当前窗口中的文档有关的信息，还提供了一些导航功能。location对象是一个很特殊的对象，
因为它既是window对象的属性，也是document对象的属性；换句话说，window.location和document.location引用的是同一个对象。
location对象的用处不只表现在它保存着当前文档的信息，还表现在它将URL解析为独立的片段，让开发人员可以通过不同的属性访问这些片段

## 3、常用属性
+ hash 从井号 (#) 开始的 URL，如果地址里没有“#”，则返回空字符串，可以用于保存网页状态
+ host 主机名+端口号
+ hostname 主机名
+ port 端口号
+ href 完整的 URL，在浏览器的地址栏上怎么显示它就怎么返回
+ pathname URL的路径部分。
+ protocol URL的协议，取值为 'http:','https:','file:' 等等
+ search 从问号 (?) 开始的 URL（查询部分）

## 4、常用方法
+ assign()  
   + 使用location对象可以通过很多方式来改变浏览器的位置,最常用的方式就是使用，assign()方法并为它传递一个参数URL：  
      + location.assign('https://www.google.com');  
      + 这样就可以打开新URL，并在浏览器的历史记录中生成一条记录
   + 如果将location.href和window.location设置为一个URL值。也会以该值调用assign()方法。下面的两行代码的效果是一样的：  
      + window.location = 'https://www.google.com';  
      + location.href = 'https://www.google.com';
+ reload() 
   + 重新加载当前文档，相当于按浏览器上的“刷新”(IE)或“Reload”(Netscape)键
      + location.reload();       //从页面加载
      + location.reload(true);   //从服务器加载
   + 位于reload()方法后的代码可能会也可能不会执行，这要取决于网络延迟或系统资源等因素，因此，最好将reload()放在代码的最后一行   
+ replace() 
   + 当通过上述任何一种方式修改URL后，浏览器的历史记录中就会生成一条新记录。要禁用这种行为可以使用replace()方法，这个方法只接收一个参数，
既要导航到的URL，结果虽然导致浏览器位置变化，但不在历史记录中生成新记录。在调用replace()方法后，用户不能回到前一个页面，此时后退按
钮将处于禁用状态：
      + location.replace('https://www.google.com');

## 5、编码/解码
+ 通过 location 获取的URL可能是被编码后的，获取之后需要解码
+ 在js中可以使用三种方式编码
   + escape()
   + encodeURL()
   + encodeURIComponent()
+ 对应三种方式解码 
   + unescape()
   + unescapeURL()
   + decodeURIComponent()
+ 三者区别
   + escape 和 unescape  
      +	原理：对除ASCII字母、数字、标点符号 @  *  _  +  -  .  / 以外的其他字符进行编码  
	  + 编码：escape('http://www.baidu.com?name=zhang@xiao@jie&order=1')
	  + 结果："http%3A//www.baidu.com%3Fname%3Dzhang@xiao@jie%26order%3D1"  
      + 解码：unescape("http%3A//www.baidu.com%3Fname%3Dzhang@xiao@jie%26order%3D1")  
      + 结果："http://www.baidu.com?name=zhang@xiao@jie&order=1"  
   + encodeURI 和 decodeURI  
      + 原理：返回编码为有效的统一资源标识符 (URI) 的字符串，不会被编码的字符：  
      	! @ # $ & * ( ) = : / ; ? + '  
      	encodeURI()是Javascript中真正用来对URL编码的函数  
      + 编码：encodeURI('http://www.baidu.com?name=zhang@xiao@jie&order=1')  
      + 结果："http://www.baidu.com?name=zhang@xiao@jie&order=1"  
      + 解码：decodeURI("http%3A//www.baidu.com%3Fname%3Dzhang@xiao@jie%26order%3D1")  
      + 结果："http%3A//www.baidu.com%3Fname%3Dzhang@xiao@jie%26order%3D1"  
   + encodeURIComponent 和 decodeURIComponent  
      + 原理：对URL的组成部分进行个别编码，而不用于对整个URL进行编码  
      + 编码：encodeURIComponent('http://www.baidu.com?name=zhang@xiao@jie&order=1')  
      + 结果："http%3A%2F%2Fwww.baidu.com%3Fname%3Dzhang%40xiao%40jie%26order%3D1"  
      + 解码：decodeURIComponent("http%3A%2F%2Fwww.baidu.com%3Fname%3Dzhang%40xiao%40jie%26order%3D1")  
      + 结果："http://www.baidu.com?name=zhang@xiao@jie&order=1"  
      
## 6、参考
+ [https://www.jb51.net/article/65963.htm](https://www.jb51.net/article/65963.htm){:target="_blank"}
+ [https://blog.csdn.net/luofeng457/article/details/70214101](https://blog.csdn.net/luofeng457/article/details/70214101){:target="_blank"}
+ [https://www.cnblogs.com/luckyuns/p/6396701.html](https://www.cnblogs.com/luckyuns/p/6396701.html){:target="_blank"}
+ [https://www.cnblogs.com/z-one/p/6542955.html](https://www.cnblogs.com/z-one/p/6542955.html){:target="_blank"}