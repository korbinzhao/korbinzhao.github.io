---
layout: post
title: 通过 Paint Timing API 提升性能
description: 衡量性能非常重要，衡量终端用户的感知性能更为重要。用户感知性能则主要受这两个指标影响。
categories: 
- 前端开发
- 性能优化

tags: 
- 前端开发
- 性能优化
---

## 介绍
传统上，页面加载时间是前端性能的一个重要指标。商业web性能指标板会展示后端方面颗粒级的性能指标，数据库查询时间、模板编译、服务器相应时间等。

然而，在客户端，性能信息的收集是受限的。部分取决于浏览器能够提供的性能指标和API。具有讽刺意义的是，一些不标准的性能统计技术反而会给性能带来消极影响。得益于新的性能API的出现，这一情况也在悄然改变。新的性能API如下：
* User Timing API 
* Navigation Timing API
* Network Information API
* Resource Timing API
* Paint Timing API

接下来，我们谈谈在 Paint Timing API 上。Paint Timing API 中有2个重要指标：First Paint 和 First Conentful Paint。

渲染指标回答了访问我们页面的两个有用的问题：
* 是否有什么发生？（First paint）
* 是否有用？（First contentful paint）

衡量性能非常重要，衡量终端用户的感知性能更为重要。用户感知性能则主要受这两个指标影响。

## First Paint
屏幕中发生的第一次渲染，这个指标不包含默认背景渲染，但是包含任何用户定义的背景渲染，也就是第一个像素被渲染到屏幕上的时间点。

## First Contentful Paint
第一个dom节点被渲染到屏幕上的时间点，这个dom节点可以是任何文本、图片、canvas或者SVG等等。

举个栗子。以twitter和nest为例，二者的 First Contentful Paint 时间点页面的样子如下。
![](https://www.sitepen.com/blog/wp-content/uploads/2017/09/image_0-1.png)
![](https://www.sitepen.com/blog/wp-content/uploads/2017/09/image_1-1.png)

## 通过代码理解二者差异
[demo github 地址](https://github.com/SitePen/paint-timing-blog-examples/blob/6c466715ad07659fc04541cfada1542677540879/paint-delay/index.html)

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Paint delay example</title>
    <script>
        const observer = new window.PerformanceObserver(list => {
            list.getEntries().forEach(({name, startTime}) => {
                console.log({name, startTime});
            });
        });
 
        observer.observe({
            entryTypes: ['paint']
        });
 
        function sleep(ms = 1000) {
            return new Promise(resolve => setTimeout(resolve, ms));
        }
 
        // This triggers first-paint
        sleep().then(() => document.body.style.backgroundColor = 'lightgrey');
 
        // This triggers first-contentful-paint
        sleep(2000).then(() => document.body.innerHTML += '<p>Hi there!</p>');
    </script>
</head>
<body>
 
</body>
</html>
```

查看demo之前，需要注意下  PerformanceObserver() API 的用法。这个API提供了丰富的性能指标。

## 使用 JS 获取渲染指标
### 方法1：查询渲染指标
Chrome DevTools 中输入：

```
performance.getEntriesByType('paint')
```
### 方法2：监听渲染指标
```
const observer = new PerformanceObserver(list => {
    // `list` provides access to performance metrics
});
 
observer.observe({entryTypes: ['paint']});
```

## 参考文献
* https://www.sitepen.com/blog/2017/10/06/improving-performance-with-the-paint-timing-api/