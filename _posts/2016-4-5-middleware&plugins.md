---
layout: post
title: nodejs 中间件 & 插件
description: 中间件和插件的概念与区别
categories: 
- 前端开发
tags: 
- 前端开发
---

## 中间件
中间件是一种独立的系统软件或服务程序，分布式应用软件借助这种软件在不同的技术之间共享资源。中间件位于客户机/ 服务器的操作系统之上，管理计算机资源和网络通讯。是连接两个独立应用程序或独立系统的软件。相连接的系统，即使它们具有不同的接口，但通过中间件相互之间仍能交换信息。执行中间件的一个关键途径是信息传递。通过中间件，应用程序可以工作于多平台或 OS 环境。例如：IBM MQSeries，CICS/TXSeries

### 如何理解 "中间件" ?

我的理解是这样的，中间件就是类似于一个过滤器的东西，在客户端和应用程序之间的一个处理请求和响应的的方法。

如果把一个http处理过程比作是污水处理，中间件就像是一层层的过滤网。每个中间件在http处理过程中通过改写request或（和）response的数据，状态，实现了特定的功能。

以 koa 中间件为例：

Koa的中间件很像Express的中间件，也是对HTTP请求进行处理的函数，但是必须是一个Generator函数。而且，Koa的中间件是一个级联式（Cascading）的结构，也就是说，属于是层层调用，第一个中间件调用第二个中间件，第二个调用第三个，以此类推。上游的中间件必须等到下游的中间件返回结果，才会继续执行，这点很像递归。

	var koa = require('koa');
	var http = require('http');
	var app = koa();

	app.use(function *(next){
 	 console.log('>> one');
  	 yield next;
  	 console.log('<< one');
	});

	app.use(function *(next){
  	 console.log('>> two');
 	 this.body = 'two';
  	 yield next;
 	 console.log('<< two');
	});

	app.use(function *(next){
  	  console.log('>> three');
  	  yield next;
  	  console.log('<< three');
	});

	http.createServer(app.callback()).listen(3000);

	console.log('server start at port 3000');

## 插件
插件是一种遵循一定规范的应用程序接口编写出来的程序。很多软件都有插件，插件有无数种。例如在IE中，安装相关的插件后，WEB浏览器能够直接调用插件程序，用于处理特定类型的文件。IE浏览器常见的插件例如：Flash插件、RealPlayer插件、MMS插件、MIDI五线谱插件、ActiveX插件等等；再比如Winamp的DFX，也是插件。还有很多插件都是程序员新开发的。

## 组件
组件和插件的区别是，插件是属于程序接口的程序，组件在ASP中就是控件、对象，ASP/IIS的标准安装提供了11个可安装组件。ASP的FSO组件，编程的朋友都如雷贯耳吧，它就是最常用的Scripting.FileSystemObject对象。

## 套件
套件（package）Java提供package机制 ，它就像是一个管理容器，可以将您所定义的名称区隔管理在package下，而不会有相互冲突的发生，例如您定义了一个dimension2d与dimension3d的package，在它们之下都有一个Point类别，但由于属于不同的package，所以这两个名称并不会有所冲突。

## 参考资料
1. [大熊君大话NodeJS之------Connect中间件模块（第一季）](http://www.cnblogs.com/bigbearbb/p/4221378.html)
2. [中间件、插件、组件、套件之间的差别](http://www.cnblogs.com/enshrineZither/p/4093872.html)

