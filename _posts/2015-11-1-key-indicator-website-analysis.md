---
layout: post
title: 网站分析指标
description: 我们在做一个网站或网页分析时，肯定会用到UV、PV、跳失率等概念。那么网站分析的一些关键指标到底具体指什么呢？
categories: 
- 产品经理
tags: 
- 产品经理
---



## 1. UV（Unique Vistor）

UV是指唯一的访问者。00:00-24:00内相同的UV只被计算一次。

### 网站分析工具如何辨别UV

* 通过服务器
服务器从浏览器接收到的每次请求都会包含IP、请求时间、浏览器版本、操作系统版本等很多信息，服务器可以根据某些共同特征，来判断是否是同一个UV。不过根据IP判断UV并不十分准确，尤其是在公用IP的局域网环境下。
* 通过cookie
浏览器第一次访问某个网站服务器时，服务器会给浏览器发送一个cookie，这个cookie会分配给你一个独一无二的编号，并且会记录你访问服务器的信息，例如访问时间、访问的网页、是否登录、登录的账号信息等。当浏览器再一次访问服务器时，服务器获取到cookie，并对里面的信息进行更新，但是给你的编号是不会变的。cookie中拥有相同编号的两个请求会被认定为同一个visitor。这个方法比上一个更精确些，但是在浏览器禁用cookie或者用户经常删除cookie的情况下，这种方式也将失效。

## 2. PV（Page View）

PV（page view），即页面浏览量，或点击量。一个PV指的是一次从网站下载一个页面的请求（A page view or page impression is a request to load a single page of an Internet site.）。

##3. Bounce Rate

Bounce Rate，指的是在对一个页面的所有访问中，只浏览却没有发生任何点击的访问所占的比率。这种跳失一般发生在用户误点跳转链接或是进入网页后发现内容非常不感兴趣时。

用一个公式来表示Bounce Rate，如下：

	Omniture的计算方法：
	
	Bounce Rate = Single Page Visits / Total Visits
	
不过Bounce Rate存在另一种计算方法：

	Google Analytics等工具的计算方法：
	
	Bounce Rate = Single PV Visits / Total Visits 
	
Page Visits 和 PV Visits的主要区别在于在用户刷新时，对visits的统计是否会+1. 举例说明，用户进入页面后在没有任何操作情况下，刷新一次页面Page Visits不会+1，而PV就会+1，使得PV变为2.

这两种计算方法的具体区别详见[网站分析的最基本度量]。

## 4. 跳失率

业界有『exit rate』和『bounce rate』两个概念，其中『exit rate』指该网页是会话中『最后一页』的浏览占此网页总浏览量的百分比，而『bounce rate』是指该网页是会话中“唯一网页”的会话占从此网页开始的所有会话的百分比。


## PS：

当然，以上这些概念只是行业通用的一些行业，有些甚至没有统一的界定和标准。每个公司都有自己的一套界定规范。如果你是身在职场的孩子，不妨详细了解下本公司对各网站分析指标的详细界定。

[Joebon]:    http://korbinzhao.github.io  "Joebon"

[网站分析的最基本度量]: http://www.woshipm.com/operate/83233.html
