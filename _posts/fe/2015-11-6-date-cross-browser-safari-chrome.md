---
layout: post
title: Date对象的浏览器兼容性问题
description: 介绍一个能够解决Date对象的浏览器兼容问题的简单方法。
category: fe
---

## new Date()、Date.parse()方法在浏览器中的兼容性问题
    
   Date在不同浏览器中对于传入的时间字符串的格式要求是不一样的。比如在chrome浏览器的控制台中输入以下内容，会得到相应结果：
   
    Date.parse('2015-11-11 00:00:00')
    //->1447171200000
    Date.parse('2015/11/11 00:00:00')
    //->1447171200000
    new Date('2015-11-11 00:00:00')
    //->Wed Nov 11 2015 00:00:00 GMT+0800 (CST)
    new Date('2015/11/11 00:00:00')
    //->Wed Nov 11 2015 00:00:00 GMT+0800 (CST)
    
  可以发现，chrome对于'-'和'/'分割日期的形式都是支持的，能够返回正确的毫秒数或者时间格式。
  
  下面在safari浏览器中输入相同内容，查看对应结果：
    
    > Date.parse('2015-11-11 00:00:00')
    < NaN = $1
    > Date.parse('2015/11/11 00:00:00')
    < 1447171200000 = $2
    > new Date('2015-11-11 00:00:00')
    < Invalid Date = $3
    > new Date('2015/11/11 00:00:00')
    < Wed Nov 11 2015 00:00:00 GMT+0800 (CST) = $4
    
   可以发现，safari仅对'/'分割日期的形式支持，对'-'分割日期的形式并不支持。
    
  通过以上实验，我们可以观察出chrome和safari对于不同格式的时间字符串的支持情况。chrome同时支持'-'和'/'分割日期的时间字符串；safari不支持'-'分割日期的时间字符串。
 
## 一个简单有效的解决方法
    
  下面是一个比较好的解决方案：
  
    var arr = "2010-03-15 10:30:00".split(/[- / :]/),
        date = new Date(arr[0], arr[1]-1, arr[2], arr[3], arr[4], arr[5]);
      
      console.log(date);
      //-> Mon Mar 15 2010 10:30:00 GMT+0000 (GMT Standard Time)
  
   该方法为先将时间字符串用split方法进行分割拼装为一个数组，再将每个数组的项作为传参传入new Date()方法，从而将不同形式的字符串转换成有效的时间。
   该方法对于所有浏览器生效。



## 参考资料：

[http://stackoverflow.com/questions/5324178/javascript-date-parsing-on-iphone](http://stackoverflow.com/questions/5324178/javascript-date-parsing-on-iphone)

[Joebon]:    http://joebon.tk  "Joebon"


    