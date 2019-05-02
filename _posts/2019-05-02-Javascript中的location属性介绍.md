---
layout: post
title:  "Javascript中的location属性介绍"
date:   2019-05-02 17:39:00 +0800
categories: 笔记
tags: Javascript location
---


Javascript中的location属性介绍
### 1、location对象
location是最有用的DOM对象之一，它提供了与当前窗口中的文档有关的信息，还提供了一些导航功能。location对象是一个很特殊的对象，
因为它既是window对象的属性，也是document对象的属性；换句话说，window.location和document.location引用的是同一个对象。
location对象的用处不只表现在它保存着当前文档的信息，还表现在它将URL解析为独立的片段，让开发人员可以通过不同的属性访问这些片段。
下标列出了location对象的所有属性（省略了每个属性前面的location前缀）：
Alt text

##### 1.1、查询字符串参数
虽然上面的属性可以访问到location对象的大多数信息，但是其中访问URL包含的查询字符串的属性并不方便。尽管location.search返回从
问号到URL末尾的所有内容，但是却没有办法逐个访问其中的每个查询字符串参数。因此可以创建一个函数，用以解析查询字符串，然后返回包含
所以参数的一个对像。

##### 1.2、位置操作
使用location对象可以通过很多方式来改变浏览器的位置。最常用的方式就是使用
assign()方法并为它传递一个参数URL：
```
location.assign('http://www.wrox.com');
```
这样就可以打开新URL，并在浏览器的历史记录中生成一条记录。如果将location.href和window.location设置为一个URL值。也会以该值调用
assign()方法。下面的两行代码的效果是一样的：
```
window.location = 'http://www.wrox.com';
location.href = 'http://www.wrox.com';
```
最常用的是设置location.href属性。
修改location对象的其他属性也可以改变当前加载的页面：
```
//假设初始值URL为：http://www.wrox.com/WileyCDA
//将URL修改为：http://www.wrox.com/WileyCDA/#section
location.hash = '#section';
//将URL修改为：http://www.wrox.com/WileyCDA/?q=javascript
location.search = '?q=javascript';
//将URL修改为：http://www.wrox.com/WileyCDA/
location.hostname = 'www.wrox.com';
//将URL修改为：http://www.wrox.com/mydir
location.pathname = 'mydir';
//将URL修改为：http://www.wrox.com:8080/WileyCDA/
location.port = 8080;
// 每次修改location的属性（hash除外）都会以新URL重新加载
```
当通过上述任何一种方式修改URL后，浏览器的历史记录中就会生成一条新记录。要禁用这种行为可以使用replace()方法，这个方法只接收一个参数，
既要导航到的URL，结果虽然导致浏览器位置变化，但不在历史记录中生成新记录。在调用replace()方法后，用户不能回到前一个页面，此时后退按
钮将处于禁用状态：
```
location.replace('www.wrox.com');
```
与为重有关的最后一个方法是reload()，作用是重新加载当前显示的页面。如果调研那个reload()不传递任何参数，页面就以最有效的方式重载，
也就是从浏览器缓存中冲洗加载；如果想强制从服务器重新加载，则需要：
```
location.reload();       //从页面加载
location.reload(true);   //从服务器加载
```
位于reload()方法后的代码可能会也可能不会执行，这要取决于网络延迟或系统资源等因素，因此，最好将reload()放在代码的最后一行。