---
layout: post
title: 前端模块化规范 - CommonJS、AMD & CMD
description: 前端为什么会出现模块化这个概念，主要原因是随着前端项目的复杂性和项目团队规模的扩大，命名冲突和文件依赖成为影响前端开发效率的两个重要问题。模块化规范的另一个目的是提高JS程序的可移植性。
category: fe
---

前端为什么会出现模块化这个概念，主要原因是随着前端项目的复杂性和项目团队规模的扩大，命名冲突和文件依赖成为影响前端开发效率的两个重要问题。模块化规范的另一个目的是提高JS程序的可移植性。

## CommonJS

官方JavaScript规范定义的API主要是为了构建基于浏览器端的应用程序。在服务器端，并没有官方的API定义。

CommonJS定义了许多普通应用程序（主要指非浏览器应用）使用的API，它的终极目标是提供一个类似Python、Ruby和Java的标准库，从而开发者可以使用CommonJS API编写可以运行在不同JS解释器和不同主机环境的应用程序。

* NodeJS和CommonJS之间的关系

	CommonJS是一种规范，NodeJS是这种规范的实现。JavaScript作为本地变成语言这种特殊应用程序，也是随着NodeJS的火热而进入大众的视野。
	
* 浏览器加载 CommonJS 模块的原理与实现

	npm的模块都是JS语言写的，但是浏览器用不了，因为浏览器不支持CommonJS格式。要想让浏览器用上这些模块，必须转换格式。
	
	浏览器不兼容CommonJS的根本原因，在于缺少四个Node.js环境的变量。
	
		module
		exports
		require
		global

	只要能够提供这四个变量，浏览器就能加载CommonJS模块。
	
* Browserify的实现

知道了原理，就能做出工具了。Browserify（require('modules') in the browser）是目前最常用的CommonJS格式转换的工具。

举例：
main.js模块加载foo.js模块

	// foo.js
	module.exports = function(x) {
	  console.log(x);
	};
	
	// main.js
	var foo = require("./foo");
	foo("Hi");
	
使用下面的命令，就能将main.js转为浏览器可用的格式。

	$ browserify main.js > compiled.js
	
Browserify到底做了什么？安装一下browser-unpack，就能看清楚了。

	$ npm install browser-unpack -g
	
然后，将前面生成的compile.js解包。

	$ browser-unpack < compiled.js
	
	[
	  {
	    "id":1,
	    "source":"module.exports = function(x) {\n  console.log(x);\n};",
	    "deps":{}
	  },
	  {
	    "id":2,
	    "source":"var foo = require(\"./foo\");\nfoo(\"Hi\");",
	    "deps":{"./foo":1},
	    "entry":true
	  }
	]
	
可以看到，browerify将所有模块放入一个数组，id属性是模块的编号，source属性是模块的源码，deps属性是模块的依赖。

因为main.js里面加载了foo.js，所以deps属性就制定./foo对应的1号模块。执行的时候，浏览器遇到require('./foo')语句，就自动执行1号模块的source属性，并就爱那个执行后的module.exports属性值输出。

## AMD

AMD（异步模块定，Asynchronous Module Definition）格式总体的目标是为现在的开发者提供一个可用的模块化JS解决方案。

CommonJS通过引入模块化使得在服务器端编写JS应用成为可能，但是由于一个重大的局限，使得CommonJS规范不适用与浏览器环境。举例如下。

	var math = require('math');
	　　math.add(2, 3);
第二行math.add(2, 3)，在第一行require('math')之后运行，因此必须等math.js加载完成。也就是说，如果加载时间很长，整个应用就会停在那里等。

这对服务器端不是一个问题，因为所有的模块都存放在本地硬盘，可以同步加载完成，等待时间就是硬盘的读取时间。但是，对于浏览器，这确实一个大问题，因为模块都放在服务器端，等待时间取决于网速的快慢，可能要等很长时间，浏览器处于“假死”状态。

因此，浏览器端的模块，不能采用“同步加载”（synchronous），只能采用“异步加载”（asynchronous）。这就是AMD规范诞生的背景。

AMD是“Asynchronous Module Definition”的缩写，意思就是“异步模块定义”。它采用异步方式加载模块，模块的加载不影响它后面语句的运行。所有依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行。

AMD也采用require()语句加载模块，但是不同于CommonJS，他要求两个参数：

	require([module], callback);
	
第一个参数[module]，是一个数组，里面的成员就是要加载的模块；第二个参数callback，则是加载成功之后的回调函数。

## CMD

CMD（Common Module Definition）模块定义规范，是@玉伯提出的一个类似于AMD的规范，该规范明确了模块的基本书写格式和基本交互规则。

### AMD和CMD的区别

下面来看下@玉伯给出的官方解释。

AMD和CMD等规范的目的都是为了JS的模块化开发，特别是在浏览器端的。目前这些规范的实现都能达成浏览器端模块化开发的目的。

区别在于：

1. 对于依赖的模块，AMD是提前执行，CMD是延迟执行。不过RequireJS从2.0开始，也改成了可以延迟执行（根据写法不同，处理方式不同）。CMD推崇as lazy as possible。
2. CMD推崇依赖就近，AMD推崇依赖前置。如下：


		// CMD
		define(function(require, exports, module) {   
			var a = require('./a')   
			a.doSomething()   
			// 此处略去 100 行   
			var b = require('./b') // 依赖可以就近书写   
			b.doSomething()   
			// ... 
		})
		
		// AMD 默认推荐的是
		define(['./a', './b'], function(a, b) {  
			// 依赖必须一开始就写好    
			a.doSomething()    
			// 此处略去 100 行    
			b.doSomething()    
			...
		})

	虽然AMD也支持CMD的写法，同时还支持require作为依赖项传递，但RequireJS官方文档里默认上面的模块定义写法。

3. AMD的API默认是一个当多个用，CMD的API严格区分，推崇职责单一。比如AMD里，require分全局require和局部require，都叫require。CMD里，没有全局require，而是根据模块系统的完备性，提供seajs.use来实现模块系统的加载启动。CMD里，每个API都简单纯粹。





## 参考资料

1. [什么是CommonJS？](http://www.cnblogs.com/fullhouse/archive/2011/07/15/2107416.html)
2. [JavaScript模块化开发（二）——CommonJS规范](http://www.feeldesignstudio.com/2013/09/javascript-module-pattern-commonjs/)
3. [浏览器加载 CommonJS 模块的原理与实现](http://www.ruanyifeng.com/blog/2015/05/commonjs-in-browser.html)
4. [Javascript模块化编程（二）：AMD规范](http://www.ruanyifeng.com/blog/2012/10/asynchronous_module_definition.html)
3. [AMD 和 CMD 的区别有哪些？- 知乎](http://www.zhihu.com/question/20351507)