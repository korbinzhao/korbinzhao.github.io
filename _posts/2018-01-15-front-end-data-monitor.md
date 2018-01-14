---
layout: post
title: 前端数据监控到底在监控什么？
description: 前端数据监控一般分为性能数据监控和线上异常监控。本文对这两块数据的监控原理和方法进行整理说明。
categories: 
- 前端开发
- 数据监控

tags: 
- 前端开发
- 数据监控

---


前端数据监控一般分为性能数据监控和线上异常监控。本文对这两块数据的监控原理和方法进行整理说明。

## 性能数据

### 统计方案
* 代码监控
  * 将监控代码注入到页面中，手动计算时间差或者使用浏览器API进行数据统计。 
* 工具监控
  * 不将统计代码注入到页面中，一般借助虚拟机对页面进行性能数据分析。
  
  
|类型	|优点	|缺点|	示例|
|---|---|---|---|
|非侵入式|	指标齐全、客户端主动监测、竞品监控|	无法知道性能影响用户数、采样少容易失真、无法监控复杂应用与细分功能|	Pagespeed、PhantomJS、UAQ|
|侵入式	|真实海量用户数据、能监控复杂应用与业务功能、用户点击与区域渲染|	需插入脚本统计、网络指标不全、无法监控竞品|	DP 、Google 统计|

在进行性能数据监控之前，先要明确页面从用户开始访问到页面加载完成经历的时间阶段。

### 时间阶段


![](https://gw.alicdn.com/tfs/TB1OEKUkyqAXuNjy1XdXXaYcVXa-1886-944.png)


按触发顺序排列所有属性：(更详细标准的解释请参看：[W3C Editor's Draft](https://w3c.github.io/navigation-timing/#introduction))

* navigationStart:在同一个浏览器上下文中，前一个网页（与当前页面不一定同域）unload 的时间戳，如果无前一个网页 unload ，则与 fetchStart 值相等
* unloadEventStart:前一个网页（与当前页面同域）unload 的时间戳，如果无前一个网页 unload 或者前一个网页与当前页面不同域，则值为 0
* unloadEventEnd:和 unloadEventStart 相对应，返回前一个网页 unload 事件绑定的回调函数执行完毕的时间戳
* redirectStart:第一个 HTTP 重定向发生时的时间。有跳转且是同域名内的重定向才算，否则值为 0
* redirectEnd:最后一个 HTTP 重定向完成时的时间。有跳转且是同域名内的重定向才算，否则值为 0
* fetchStart:浏览器准备好使用 HTTP 请求抓取文档的时间，这发生在检查本地缓存之前
* domainLookupStart:DNS 域名查询开始的时间，如果使用了本地缓存（即无 DNS 查询）或持久连接，则与 fetchStart 值相等
* domainLookupEnd:DNS 域名查询完成的时间，如果使用了本地缓存（即无 DNS 查询）或持久连接，则与 fetchStart 值相等
* connectStart:HTTP（TCP） 开始建立连接的时间，如果是持久连接，则与 fetchStart 值相等,如果在传输层发生了错误且重新建立连接，则这里显示的是新建立的连接开始的时间
* connectEnd:HTTP（TCP） 完成建立连接的时间（完成握手），如果是持久连接，则与 fetchStart 值相等,如果在传输层发生了错误且重新建立连接，则这里显示的是新建立的连接完成的时间

  注意:这里握手结束，包括安全连接建立完成、SOCKS 授权通过

* secureConnectionStart:HTTPS 连接开始的时间，如果不是安全连接，则值为 0
* requestStart:HTTP 请求读取真实文档开始的时间（完成建立连接），包括从本地读取缓存,连接错误重连时，这里显示的也是新建立连接的时间
* responseStart:HTTP 开始接收响应的时间（获取到第一个字节），包括从本地读取缓存
* responseEnd:HTTP 响应全部接收完成的时间（获取到最后一个字节），包括从本地读取缓存
* domLoading:开始解析渲染 DOM 树的时间，此时 Document.readyState 变为 loading，并将抛出 readystatechange 相关事件
* domInteractive:完成解析 DOM 树的时间，Document.readyState 变为 interactive，并将抛出 readystatechange 相关事件

  注意:只是 DOM 树解析完成，这时候并没有开始加载网页内的资源

* domContentLoadedEventStart:DOM 解析完成后，网页内资源加载开始的时间,文档发生 DOMContentLoaded事件的时间
* domContentLoadedEventEnd:DOM 解析完成后，网页内资源加载完成的时间（如 JS 脚本加载执行完毕），文档的DOMContentLoaded 事件的结束时间
* domComplete:DOM 树解析完成，且资源也准备就绪的时间，Document.readyState 变为 complete，并将抛出 readystatechange 相关事件
* loadEventStart:load 事件发送给文档，也即 load 回调函数开始执行的时间,如果没有绑定 load 事件，值为 0
* loadEventEnd:load 事件的回调函数执行完毕的时间,如果没有绑定 load 事件，值为 0

DOMContentLoaded 和 load 事件的区别，详见 [DOMContentLoaded与load的区别](https://www.cnblogs.com/caizhenbo/p/6679478.html)。

### 统计方法

#### Navigation Timing API

可以直接使用Navigation Timing接口来获取统计起点以及加载过程中的各个阶段耗时。[window.performance](https://www.w3.org/TR/navigation-timing-2/) 是W3C性能小组引入的新的API，目前IE9以上的浏览器都支持。

常用计算：
* DNS查询耗时 ：domainLookupEnd - domainLookupStart
* TCP链接耗时 ：connectEnd - connectStart
* request请求耗时 ：responseEnd - responseStart
* 解析dom树耗时 ： domComplete - domInteractive
白屏时间 ：responseStart - navigationStart
* domready时间(用户可操作时间节点) ：domContentLoadedEventEnd - navigationStart
* onload时间(总下载时间) ：loadEventEnd - navigationStart

####  Resource timing API

Resource timing API是用来统计静态资源相关的时间信息，详细的内容请参考[W3C Resource timing](https://www.w3.org/TR/resource-timing/)。这里我们只介绍performance.getEntries方法


### 关键指标及计算方法
  * 白屏时间（first paint time）- 用户从打开页面到页面开始有内容呈现为止
  * 首屏时间 - 用户从打开页面到首屏内所有内容都呈现出来所花费的时间
  * 用户可操作时间（dom interactive） - 用户从打开页面到可以进行正常点击、输入等操作的时间
  * 总下载时间 - 用户从打开页面到页面所有资源都加载完毕并呈现出来所化的时间

#### 确定统计起点
我们需要在用户输入 URL 或者点击链接的时候就开始统计，因为这样才能衡量用户的等待时间。如果你的用户高端浏览器占比很高，那么可以直接使用Navigation Timing接口来获取统计起点以及加载过程中的各个阶段耗时。

#### 白屏时间

白屏时间是用户首次看到内容的时间，也叫做首次渲染时间，chrome 高版本有 firstPaintTime 接口来获取这个耗时，但大部分浏览器并不支持，必须想其他办法来监测。仔细观察 WebPagetest 视图分析发现，白屏时间出现在头部外链资源加载完附近，因为浏览器只有加载并解析完头部资源才会真正渲染页面。基于此我们可以通过获取头部资源加载完的时刻来近似统计白屏时间。尽管并不精确，但却考虑了影响白屏的主要因素：首字节时间和头部资源加载时间。

如何统计头部资源加载呢？我们发现头部内嵌的 JS 通常需等待前面的 JS\CSS 加载完才会执行，是不是可以在浏览器 head 内底部加一句 JS 统计头部资源加载结束点呢？可以通过一个简单的示例进行测试：

```
<!DOCTYPE HTML>
<html>
    <head>
        <meta charset="UTF-8"/>
    <script>
      var start_time = +new Date; //测试时间起点，实际统计起点为 DNS 查询
    </script>
    <!-- 3s 后这个 js 才会返回 -->
    <script src="script.php"></script>  
    <script>
      var end_time = +new Date; //时间终点
      var headtime = end_time - start_time; //头部资源加载时间    
      console.log(headtime);
    </script>
    </head> 
    <body>     
    <p>在头部资源加载完之前页面将是白屏</p>
    <p>script.php 被模拟设置 3s 后返回，head 底部内嵌 JS 等待前面 js 返回后才执行</p>
    <p>script.php 替换成一个执行长时间循环的 js 效果也一样</p>  
    </body>
</html>
```

经测试发现，统计的头部加载时间正好跟头部资源下载时间相近，而且换成一个执行时间很长的 JS 也会等到 JS 执行完才统计。说明此方法是可行的(具体原因可查看浏览器渲染原理及 JS 单线程相关介绍)。

除了上述方法，还可以采用 navigation timing api 来获取白屏时间。该方法的缺点是浏览器兼容性较差，在 safari 等浏览器中无法使用。

```
 var firstPaint = 0;

  // Chrome
  if (window.chrome && window.chrome.loadTimes) {
      // Convert to ms
      firstPaint = window.chrome.loadTimes().firstPaintTime * 1000;
      api.firstPaintTime = firstPaint - timing.navigationStart;
  }
  // IE
  else if (typeof timing.msFirstPaint === 'number') {
      firstPaint = timing.msFirstPaint;
      api.firstPaintTime = firstPaint - timing.navigationStart;
  }
  // Firefox
  // This will use the first times after MozAfterPaint fires
  //else if (timing.navigationStart && typeof InstallTrigger !== 'undefined') {
  //    api.firstPaint = timing.navigationStart;
  //    api.firstPaintTime = mozFirstPaintTime - timing.navigationStart;
  //}

```

#### 首屏时间

在首屏渲染之前埋上处理逻辑，使用定时器不断的去检测img节点的图片。判断图片是否在首屏和加载完成，找到首屏中加载时间最慢的的图片完成的时间，从而计算出首屏时间。如果首屏有没有图片，如果没图片就用domready时间。统计流程如下：


```
首屏位置调用 API 开始统计 -> 绑定首屏内所有图片的 load 事件 -> 页面加载完后判断图片是否在首屏内，找出加载最慢的一张 -> 首屏时间
```

这是同步加载情况下的简单统计逻辑，另外需要注意的几点：

* 页面存在 iframe 的情况下也需要判断加载时间
* gif 图片在 IE 上可能重复触发 load 事件需排除
* 异步渲染的情况下应在异步获取数据插入之后再计算首屏
* css 重要背景图片可以通过 JS 请求图片 url 来统计(浏览器不会重复加载)
* 没有图片则以统计 JS 执行时间为首屏，即认为文字出现时间

#### 统计用户可操作

用户可操作默认可以统计domready时间，因为通常会在这时候绑定事件操作。对于使用了模块化异步加载的 JS 可以在代码中去主动标记重要 JS 的加载时间，这也是产品指标的统计方式。

```
performance.timing.domInteractive - performance.timing.navigationStart
```

#### 总下载


总下载时间默认可以统计onload时间，这样可以统计同步加载的资源全部加载完的耗时。如果页面中存在很多异步渲染，可以将异步渲染全部完成的时间作为总下载时间。

```
performance.timing.loadEventStart- performance.timing.navigationStart
```

## 线上异常
### 接口异常监控

方法很简单，在接口出错的情况下主动打点上报错误。

### JS 代码异常监控
* try catch 捕获
  
这种方案要求开发人员在编写代码的时候，在预估有异常发生的代码段使用try...catch，在发生异常时将异常信息发送给接口：

```
try{
//可能发生异常的代码段
}catch(e){
//将异常信息发送服务端
}
```

* window.onerror捕获

这种方式不需要开发人员在代码中书写大量的try...catch，通过给window添加onerror监听，在js发生异常的时候便可以捕获到错误信息，语法异常和运行异常均可被捕获到。但是window.onerror这个监听必须放在所有js文件之前才可以保证能够捕获到所有的异常信息。

*  跨域JS文件异常的捕获

目前可以说基本上所有的web产品对于js/css/image等静态资源都在服务端设置了Access-Control-Allow-Origin: *的响应头，也就是允许跨域请求。在这个环境下，只要我们在请求跨域资源的script标签上添加一个crossorigin属性即可：
```
<script src="http://static.toutiao.com/test.js" crossorigin></script>
```
这样的话，异域的test.js文件中发生异常时便可以被当前域的onerror监听捕获到详细的异常信息。

## 参考资料
1. [前端性能 -- 监控起步](https://www.cnblogs.com/chuaWeb/p/PerformanceMonitoring.html)
2. [Performance — 前端性能监控利器](https://www.cnblogs.com/bldxh/p/6857324.html)
3. [前端性能监控方案window.performance 调研(转)](https://www.cnblogs.com/sunshq/p/5312231.html)
4. [研究首屏时间？你先要知道这几点细节](http://www.alloyteam.com/2016/01/points-about-resource-loading/)
5. [初探 performance – 监控网页与程序性能](http://www.alloyteam.com/2015/09/explore-performance/)
6. [7 天打造前端性能监控系统](http://fex.baidu.com/blog/2014/05/build-performance-monitor-in-7-days/)
7. [DOMContentLoaded与load的区别](https://www.cnblogs.com/caizhenbo/p/6679478.html)