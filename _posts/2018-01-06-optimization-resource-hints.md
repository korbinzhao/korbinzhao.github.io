---
layout: post
title: 前端性能优化 - Resource Hints
description: Resource Hints 是非常好的一种性能优化方法，可以大大降低页面加载时间，给用户更加流畅的用户体验。现代浏览器使用大量预测优化技术来预测用户行为和意图，这些技术有预连接、资源与获取、资源预渲染等。
categories: 
- 前端开发
- 性能优化
- 资源预加载
tags: 
- 前端开发
- 性能优化
- 资源预加载
---

Resource Hints 是非常好的一种性能优化方法，可以大大降低页面加载时间，给用户更加流畅的用户体验。

现代浏览器使用大量预测优化技术来预测用户行为和意图，这些技术有预连接、资源与获取、资源预渲染等。

Resource Hints 的思路有如下两个：
* 当前将要获取资源的列表
* 通过当前页面或应用的状态、用户历史行为或 session 预测用户行为及必需的资源

实现Resource Hints的方法有很多种，可分为基于 link 标签的 DNS-prefetch、subresource、preload、 prefetch、preconnect、prerender，和本地存储 localStorage。

## DNS-Prefect 

DNS 预读取是一项使浏览器主动去执行域名解析的功能，其范围包括文档的所有连接，无论是图片的，CSS 的，还是 JavaScript 等其他用户能够点击的 URL。这样能够减少用户点击链接时的延迟。

DNS 请求需要的带宽非常小，但是延迟却有点高，这点在手机网络上特别明显。预读取DNS能让延迟明显减少一些，例如用户点击链接时。在某些情况下，延迟能减少一秒钟。

在某些浏览器中这个预读取的行为将会与页面实际内容并行发生（而不是串行）。正因如此，某些高延迟的域名的解析过程才不会卡住资源的加载。

这样可以极大的加速（尤其是移动网络环境下）页面的加载。在某些图片较多的页面中，在发起图片加载请求之前预先把域名解析好将会有至少5%的图片加载速度提升。

### 打开和关闭DNS预读取
可以通过在服务器端发送 X-DNS-Prefectch-Control 报头，或是在文档中使用值为 http-equiv 的 <meta> 标签：

```
<meta http-equiv="x-dns-prefetch-control" content="off">
```

可以通过将 content 的参数设置为 on 来改变设置

### 强制查询特定主机名
可以通过使用 rel 属性值为 link type 中的 dns-prefetch 的 <link> 标签来对特定域名进行预读取：

```
<link rel="dns-prefetch" href="http://www.spreadfirefox.com/">
```
强制对域名进行预读取在有的情况下很有用，比如，在网站的主页上，强制在整个网站上频繁引用的域名的预解析，及时他们不再主页本身上使用。及时主页的性能可能不受影响，这将提高整体站点的性能。

## Preconnect
现代浏览器都试着预测网站将来需要哪些连接，然后预先建立 socket 连接，从而消除昂贵的 DNS 查找、TCP握手和 TLS 往返开销。然而，浏览器并不够聪明，并不能准确预测每个网站的所有预连接目标。prefetch 允许我们告诉浏览器我们需要哪些预连接，
preconnect 不仅完成 DNS 解析，同事还会进行 TCP 握手和建立传输层协议。

```
<link rel="preconnect" href="//example.com">
<link rel="preconnect" href="//cdn.example.com" crossorigin>
```

## Prefetch
如果确定某个资源将来一定会被使用，我们可以让浏览器预先请求该资源并放入浏览器缓存中。使用方法：

```
<link rel="prefetch" href="image.png">
```
与DNS预读取不同，prefetch 预获取真正请求并下载了资源，并存储在缓存中。单预获取还依赖于一些条件，某些预获取可能会被浏览器忽略，例如一个非常缓慢的网络中获取一个庞大的字体文件。

注意：
* chrome 和 firefox 的网络面板中都有资源预获取的记录
* 预获取没有资源同源策略限制


## Subresources
除 prefetch 之外的另一种预获取方式，这种方式的预获取资源具有最高的优先级，在所有 prefetch 项之前进行：

```
<link rel="subresource" href="styles.css">
```

所以，如果资源是当前页面必须的，或者资源需要尽快可用，那么最好使用 subresource 而不是 prefetch。

## Prerender
prerender 会预先加载文档的所有资源：

```
<link rel="prerender" href="http://example.com">
```
这类似于在一个隐藏的 tab 页中打开了某一个链接，下载所有资源创建 DOM 结构、完成页面布局、应用 CSS 样式和执行 JS 脚本等。当用户真正访问该链接时，隐藏的页面就切换为可见，使页面看起来就是shunjian 加载完成一样。

需要注意的是不要滥用该特性，当你知道用户一定会点击某个链接时，才可以进行预渲染，否则浏览器将无条件地下载所有预渲染需要的资源。

## Preload

preload 预加载是区于 prefetch 的另一种 resouce hint 的方式。用于任意类型的资源。用法如下：

```
 <link rel="preload" href="style.css" as="style">
  <link rel="preload" href="main.js" as="script">
```

preload 和 prefetch 的区别在于，preload 是告诉浏览器预先请求当前页面需要的资源，具有高优先级，prefetch 是告诉浏览器用户将来可能在其他页面（非本页面）可能使用到的资源，浏览器会在空闲的情况下，预先加载这些资源到 http 缓存里。

## LocalStorage 缓存机制

前面介绍的基于 link 标签的方法存在浏览器兼容问题。目前来看，比较通用的一种资源预加载方案是利用 LocalStorage 来进行部分资源和数据的缓存。目前在各大主流互联网厂商中都有使用。

使用 LocalStorage 来做资源缓存，需要解决缓存更新机制。

但也存在一个短板，就是存在 XSS 安全隐患。因为 localStorage 的信息是可以在客户端被任意修改的。

因此，LocalStorage 缓存很有用，但是也有明显缺陷。

## 参考资料

1. [前端性能优化 - 资源预加载](http://bubkoo.com/2015/11/19/prefetching-preloading-prebrowsing/)
2. [Resource Hints](https://w3c.github.io/resource-hints/)
3. [DNS 预读取](https://developer.mozilla.org/zh-CN/docs/Controlling_DNS_prefetching)
4. [浏览器页面资源加载过程与优化](https://juejin.im/post/5a4ed917f265da3e317df515?utm_source=gold_browser_extension)
5. [localStorage的黑科技-js和css缓存机制](https://www.jianshu.com/p/0fa0bf842bbb)
6. [静态资源（JS/CSS）存储在localStorage有什么缺点？为什么没有被广泛应用？](https://www.zhihu.com/question/28467444)

