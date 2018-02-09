---
layout: post
title: 前端面试题梳理
description: 前端面试题梳理，知识体系查漏补缺。
categories: 
- 前端开发
- 面试相关

tags: 
- 前端开发
- 性能优化
---


## 面试问题汇总
http://www.cnblogs.com/bigboyLin/p/5272902.html
https://zhuanlan.zhihu.com/p/28911400

## 性能优化

### domComplete & domContentLoaded
* https://juejin.im/post/5a36499551882529c70f34b5

* domcontentloaded dom解析+执行完成
* domComplete dom解析+执行+样式、图片加载完成

## 算法
*  使用JS实现大数相加

```
function add(a, b) {
	var arrA = a.split('').reverse();
	var arrB = b.split('').reverse();
	var sum = 0,
		left = 0,
		overFlow = 0,
		arrSum = [],
		maxLen = Math.max(arrA.length, arrB.length);
	for (var i = 0; i < maxLen ; i++ ) {
		sum = overFlow;
		if (i < arrA.length) {
			sum = sum + Number(arrA[i]);
		}
		if ( i < arrB.length) {
			sum = sum + Number(arrB[i]);
		}
		left = sum % 10;
		arrSum[i] = left;
		overFlow = Math.floor(sum / 10);
	}
	if (offset !== 0) {
		arrSum[i+1] = overFlow;
	}
	return arrSum.reverse().join('');
}
```


* 深复制
参考资料：https://zhuanlan.zhihu.com/p/30330803

```
function deepCopy(p, c) {
  var c = c || {};
  for (var i in p) {
    if (typeof p[i] === 'object') {
      c[i] = (p[i].constructor === Array) ? [] : {};
      deepCopy(p[i], c[i]);
    } else {
      c[i] = p[i];
    }
  }
  return c;
}
// 调用
var copyA = {
    arrayA: [1, 2, 3, 4]
};
var copyB = deepCopy(copyA);
copyB.arrayA.push(5);
console.log(copyA.arrayA); // [1, 2, 3, 4]
```

或

```
var a = JSON.parse(JSON.stringify(b))

```

* 快速排序
http://www.ruanyifeng.com/blog/2011/04/quicksort_in_javascript.html

```
var quickSort = function(arr) {

　　if (arr.length <= 1) { return arr; }

　　var pivotIndex = Math.floor(arr.length / 2);

　　var pivot = arr.splice(pivotIndex, 1)[0];

　　var left = [];

　　var right = [];

　　for (var i = 0; i < arr.length; i++){

　　　　if (arr[i] < pivot) {

　　　　　　left.push(arr[i]);

　　　　} else {

　　　　　　right.push(arr[i]);

　　　　}

　　}

　　return quickSort(left).concat([pivot], quickSort(right));

};
```

* JavaScript实现二分法查找
  二分法查找，也称折半查找，是一种在有序数组中查找特定元素的搜索算法。查找过程可以分为以下步骤：
  （1）首先，从有序数组的中间的元素开始搜索，如果该元素正好是目标元素（即要查找的元素），则搜索过程结束，否则进行下一步。
  （2）如果目标元素大于或者小于中间元素，则在数组大于或小于中间元素的那一半区域查找，然后重复第一步的操作。
  （3）如果某一步数组为空，则表示找不到目标元素。
  参考代码:

```
   // 非递归算法
    function binary_search(arr, key) {
        var low = 0,
            high = arr.length - 1;
        while(low <= high){
            var mid = parseInt((high + low) / 2);
            if(key == arr[mid]){
                return  mid;
            }else if(key > arr[mid]){
                low = mid + 1;
            }else if(key < arr[mid]){
                high = mid -1;
            }else{
                return -1;
            }
        }
    };
    var arr = [1,2,3,4,5,6,7,8,9,10,11,23,44,86];
    var result = binary_search(arr,10);
    alert(result); // 9 返回目标元素的索引值       

    // 递归算法
    function binary_search(arr,low, high, key) {
        if (low > high){
            return -1;
        }
        var mid = parseInt((high + low) / 2);
        if(arr[mid] == key){
            return mid;
        }else if (arr[mid] > key){
            high = mid - 1;
            return binary_search(arr, low, high, key);
        }else if (arr[mid] < key){
            low = mid + 1;
            return binary_search(arr, low, high, key);
        }
    };
    var arr = [1,2,3,4,5,6,7,8,9,10,11,23,44,86];
    var result = binary_search(arr, 0, 13, 10);
    alert(result); // 9 返回目标元素的索引值  
  ```

* 统计字符串”aaaabbbccccddfgh”中字母个数或统计最多字母数。
```
var str = "aaaabbbccccddfgh";
var obj  = {};
for(var i=0;i<str.length;i++){
    var v = str.charAt(i);
    if(obj[v] && obj[v].value == v){
        obj[v].count = ++ obj[v].count;
    }else{
        obj[v] = {};
        obj[v].count = 1;
        obj[v].value = v;
    }
}
for(key in obj){
    document.write(obj[key].value +'='+obj[key].count+' '); // a=4  b=3  c=4  d=2  f=1  g=1  h=1 
}
```

## 网络
* 重定向 & 请求转发

  http://blog.csdn.net/meiyalei/article/details/2129120
  http://www.cnblogs.com/Qian123/p/5345527.html
  
  forward（转发）：
  
  是服务器请求资源,服务器直接访问目标地址的URL,把那个URL的响应内容读取过来,然后把这些内容再发给浏览器.浏览器根本不知道服务器发送的内容从哪里来的,因为这个跳转过程实在服务器实现的，并不是在客户端实现的所以客户端并不知道这个跳转动作，所以它的地址栏还是原来的地址.
  
  redirect（重定向）：
  
  是服务端根据逻辑,发送一个状态码,告诉浏览器重新去请求那个地址.所以地址栏显示的是新的URL.
  
  转发是服务器行为，重定向是客户端行为。

* HTTP的状态码有哪些？
* HTTPS是如何实现加密？

## 基础

https://zhuanlan.zhihu.com/p/30981484


### html 基础
https://github.com/jawil/blog/issues/9

* 浏览器渲染
![](https://camo.githubusercontent.com/a04b8139c9e562f06df3c32f8771168fb3bda0b4/687474703a2f2f7777312e73696e61696d672e636e2f6d773639302f6165343962613537677931666539763576696b65636a323079743063793130362e6a7067)

* 一次完整的HTTP事务是怎样的一个过程？

  基本流程：
  
  a. 域名解析
  
  b. 发起TCP的3次握手
  
  c. 建立TCP连接后发起http请求
  
  d. 服务器端响应http请求，浏览器得到html代码
  
  e. 浏览器解析html代码，并请求html代码中的资源
  
  f. 浏览器对页面进行渲染呈现给用户

### JS 基础
https://zhuanlan.zhihu.com/p/30275607

* 如何理解闭包

  https://korbinzhao.github.io/%E5%89%8D%E7%AB%AF%E5%BC%80%E5%8F%91/%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/2018/01/16/closure/ 

* 跨域请求资源的方法有哪些？
* 谈谈垃圾回收机制方式及内存管理
* 开发过程中遇到的内存泄露情况，如何解决的？

* 实现一个函数clone，可以对JavaScript中的5种主要的数据类型（包括Number、String、Object、Array、Boolean）进行值复制。

```
/**
 * 对象克隆
 * 支持基本数据类型及对象
 * 递归方法
 */

function clone(obj) {
    var o;
    switch (typeof obj) {
        case "undefined":
            break;
        case "string":
            o = obj + "";
            break;
        case "number":
            o = obj - 0;
            break;
        case "boolean":
            o = obj;
            break;
        case "object": // object 分为两种情况 对象（Object）或数组（Array）
            if (obj === null) {
                o = null;
            } else {
                if (Object.prototype.toString.call(obj).slice(8, -1) === "Array") {
                    o = [];
                    for (var i = 0; i < obj.length; i++) {
                        o.push(clone(obj[i]));
                    }
                } else {
                    o = {};
                    for (var k in obj) {
                        o[k] = clone(obj[k]);
                    }
                }
            }
            break;
        default:
            o = obj;
            break;
    }
    return o;
}
```

* 在Javascript中什么是伪数组？如何将伪数组转化为标准数组？

  伪数组（类数组）：无法直接调用数组方法或期望length属性有什么特殊的行为，但仍可以对真正数组遍历方法来遍历它们。典型的是函数的argument参数，还有像调用getElementsByTagName,document.childNodes之类的,它们都返回NodeList对象都属于伪数组。可以使用Array.prototype.slice.call(fakeArray)将数组转化为真正的Array对象。
  
  function log(){
        var args = Array.prototype.slice.call(arguments);  
  //为了使用unshift数组方法，将argument转化为真正的数组
        args.unshift('(app)');
   
        console.log.apply(console, args);
  };
  
* Javascript中callee和caller的作用？

  caller是返回一个对函数的引用，该函数调用了当前函数；
  
  callee是返回正在被执行的function函数，也就是所指定的function对象的正文。
  
* Date 处理
  https://zhuanlan.zhihu.com/p/30164902

  * 如何将Date对象输出为指定格式的字符串，如(new Date(), 'YYYY.MM.DD') => '2017.10.16'
这道题被问到的可能性还是很高的（阿里前端的面试就问了一次）

  思路就是将预先想定的格式字符串通过replace方法替换成对应的时间片段，如将YYYY替换成Date.prototype.getFullYears()的输出，当然每种形式还有不同的替换规则，但是思路大致相同。
  
  下面是一个比较稳健的替换方式。
  
  ```
  function formatDate(date, format) {
      if (arguments.length < 2 && !date.getTime) {
          format = date;
          date = new Date();
      }
      typeof format != 'string' && (format = 'YYYY年MM月DD日 hh时mm分ss秒');
      var week = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', '日', '一', '二', '三', '四', '五', '六'];
      return format.replace(/YYYY|YY|MM|DD|hh|mm|ss|星期|周|www|week/g, function(a) {
          switch (a) {
              case "YYYY": return date.getFullYear();
              case "YY": return (date.getFullYear()+"").slice(2);
              case "MM": return (date.getMonth() + 1 < 10) ? "0"+ (date.getMonth() + 1) : date.getMonth() + 1;
              case "DD": return (date.getDate() < 10) ? "0"+ date.getDate() : date.getDate();
              case "hh": return (date.getHours() < 10) ? "0"+ date.getHours() : date.getHours();
              case "mm": return (date.getMinutes() < 10) ? "0"+ date.getMinutes() : date.getMinutes();
              case "ss": return (date.getSeconds() < 10) ? "0"+ date.getSeconds() : date.getSeconds();
              case "星期": return "星期" + week[date.getDay() + 7];
              case "周": return "周" +  week[date.getDay() + 7];
              case "week": return week[date.getDay()];
              case "www": return week[date.getDay()].slice(0,3);
          }
      });
  };
  ```

### 客户端存储
https://zhuanlan.zhihu.com/p/31656937

## 正则表达式
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Regular_Expressions

* 写一个function，清除字符串前后的空格。（兼容所有浏览器）
```
function trim(str) {
    if (str && typeof str === "string") {
        return str.replace(/(^\s*)|(\s*)$/g,""); //去除前后空白符
    }
}
```
* 使用正则表达式验证邮箱格式
```  
  var reg = /^(\w)+(\.\w+)*@(\w)+((\.\w{2,3}){1,3})$/;
  var email = "example@qq.com";
  console.log(reg.test(email));  // true  
```

## web 安全
你所了解到的Web攻击技术
（1）XSS（Cross-Site Scripting，跨站脚本攻击）：指通过存在安全漏洞的Web网站注册用户的浏览器内运行非法的HTML标签或者JavaScript进行的一种攻击。
（2）SQL注入攻击
（3）CSRF（Cross-Site Request Forgeries，跨站点请求伪造）：指攻击者通过设置好的陷阱，强制对已完成的认证用户进行非预期的个人信息或设定信息等某些状态更新。

## 用户体验
swipe-tabs - 阅读源码

## 设计模式
* 对MVC、MVVM的理解
http://www.cnblogs.com/indream/p/3602348.html

* commonjs/cmd/amd
http://zccst.iteye.com/blog/2215317


## 备注
* http 状态码
```
1 消息
100 Continue
101 Switching Protocols
102 Processing
2 成功
200 OK
201 Created
202 Accepted
203 Non-Authoritative Information
204 No Content
205 Reset Content
206 Partial Content
207 Multi-Status
3 重定向
300 Multiple Choices
301 Moved Permanently
302 Move temporarily
303 See Other
304 Not Modified
305 Use Proxy
306 Switch Proxy
307 Temporary Redirect
4 请求错误
400 Bad Request
401 Unauthorized
402 Payment Required
403 Forbidden
404 Not Found
405 Method Not Allowed
406 Not Acceptable
407 Proxy Authentication Required
408 Request Timeout
409 Conflict
410 Gone
411 Length Required
412 Precondition Failed
413 Request Entity Too Large
414 Request-URI Too Long
415 Unsupported Media Type
416 Requested Range Not Satisfiable
417 Expectation Failed
421 too many connections
422 Unprocessable Entity
423 Locked
424 Failed Dependency
425 Unordered Collection
426 Upgrade Required
449 Retry With
451Unavailable For Legal Reasons
5 服务器错误（5、6字头）
500 Internal Server Error
501 Not Implemented
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
505 HTTP Version Not Supported
506 Variant Also Negotiates
507 Insufficient Storage
509 Bandwidth Limit Exceeded
510 Not Extended
600 Unparseable Response Headers
```

