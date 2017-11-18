---
layout: post
title: JS跨域问题解决方式
description: 所谓跨域，或者异源，是指主机名（域名）、协议、端口号只要有其一不同，就为不同的域（或源）。浏览器中有一个基本的策略，叫同源策略，即限制“源”自A的脚本只能操作“同源”页面的DOM。
categories: 
- 前端开发
tags: 前端开发
---

所谓跨域，或者异源，是指主机名（域名）、协议、端口号只要有其一不同，就为不同的域（或源）。浏览器中有一个基本的策略，叫同源策略，即限制“源”自A的脚本只能操作“同源”页面的DOM。以下列举几种跨域方式。

## 1.CORS

Cross-Origin Resource Sharing，跨域资源共享，简称 CORS。CORS系统定义了一种浏览器和服务器交互的方式来确定是否允许跨域请求。跨域资源共享是一种网络浏览器的技术规范，它为Web服务器定义了一种方式，允许网页从不同的域访问其资源。

CORS(Cross-Origin Resource Sharing,􏰛源􏰉源共􏰠)是 W3C 的一个工作􏰡案,定义了在必须访问跨源资源时,浏览器与服􏰁务器应该如何􏰢沟通。CORS背􏰣后的基本思想,就是使用自定义的HTTP头部让浏览器与服􏰁务器进行沟􏰢通,从而决定请求或响应是应该成功,还是应该失败。

具体如下：

* 一个简单的使用GET或POST发􏰀的请求,它没有自定义的头部,而主体内容是text/plain。在发送􏰀该请求时,需要给它􏰤加一个􏰥额外的Origin头部,其中包含请求􏰝面的源信息(协􏰦议、域名和􏰧端口),以便服􏰁务器根据这个头部信息来决定是否给予响应。下面是 Origin 头部的一个示例:

 消费者发送一个Origin报头到提供者端：Origin: http://www.a.com；

* 提供者发送一个Access-Control-Allow-Origin响应报头给消费者，如果值为“*”或Origin对应的站点，则表示同意共享资源给消费者，如果值为null或者不存在Access-Control-Allow-Origin报头，则表示不同意共享资源给消费者；

* 如果没有这个头部,或者有这个头部但源信息不匹配,浏览器就会驳􏰩回请求。正常情况下,浏览器会处理请求

注：CORS支持GET/POST请求方式

## 2.JSONP

JSONP 是 JSON with padding(填充式 JSON 或参数式 JSON)的简写。


    callback({ "name": "Nicholas" });
 
对于一段JavaScript脚本来说，其“源”与它存储的地址无关，而取决于脚本被加载的页面，例如我们在页面中使用script引入存储在其他域的脚本文件：
	
	<script src="http://www.a.com/index.js"></script>
		
这里脚本与当前页面是同源的。除了script，还有img、iframe、link等都具有跨域加载资源的能力。

Jsonp正是利用这种特性来实现跨域的：在页面中引入要跨域访问的来源，并定义回调函数处理跨域访问得到的json数据。

示例： 
    
    var script = document.createElement("script");
	  script.src = "http://freegeoip.net/json/?callback=handleResponse";
	  document.body.insertBefore(script, document.body.firstChild);
	
	  function handleResponse(response){
	    console.log("You are at IP address " + response.ip + ", which is in " +
	        response.city + ", " + response.region_name);
	  }
	  
注：JSONP只支持GET请求方式

## 3.修改document.domain来跨子域

www.a.com/1.html和a.com/2.html是不同域的，要使他们可以跨域访问，可通过修改document.domain来实现，即在两个页面中都设置：
	
	document.domain="a.com";

需要注意的是document.domain只能往父级修改，如a.com改为www.a.com是不被允许的，这也是此方法的局限性，只使用于跨子域访问。

## 4.使用window.name来跨域访问

window.name是同一浏览器窗口下载入的所有页面共享的数据字段，所有窗口都可以读写此字段的内容。

所以假设a.com要访问b.com的数据，只需要在b.com中将数据放在window.name中，然后a.com从中取出即可。此方法适用于像iframe这样的嵌套页面架构。




## 参考资料
1. [浅谈js跨域问题](http://segmentfault.com/a/1190000003784372)
2. [JavaScript高级程序设计]()


[Joebon]:    http://joebon.cc "Joebon"