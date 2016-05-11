---
layout: post
title: 无线性能优化
description: 无线性能优化可分为“加载性能”、“渲染性能”、“感知性能”。
category: fe
---



无线性能优化可分为“加载性能”、“渲染性能”、“感知性能”。

## 加载性能

我们在谈到加载性能时，一般关注的是页面的首屏加载速度。

* webp

	[WebP]，是一种支持有损压缩和无损压缩的图片文件格式。根据 Google 官方的数据，无损压缩后的 WebP 比 PNG 文件少了 26％ 的文件大小，有损压缩在具有同等SSIM索引的情况下WebP 比 JPEG 文件少25-34%的文件大小。

* iconfont

	iconfont 对于前端来说有很多优点，自由变化大小、矢量不失真、自由修改颜色、可以添加一些视觉效果，如阴影、旋转、透明度，兼容 IE6.

	可通过 [iconfont] 平台使用，使用格式如下：

		@font-face {
   			font-family: "iconfont";
   			src: url('iconfont.eot'); /* IE9*/
   			src: url('iconfont.eot?#iefix') format('embedded-opentype'), /* IE6-IE8 */
   			url('iconfont.woff') format('woff'), /* chrome、firefox */
  		 	url('iconfont.ttf') format('truetype'), /* chrome、firefox、opera、Safari, Android, iOS 4.2+*/
   			url('iconfont.svg#iconfont') format('svg'); /* iOS 4.1- */
 		}

	在移动端，目前只需引用一个 ttf 文件。

		@font-face {
   			font-family: "iconfont";
   			src: url('iconfont.ttf') format('truetype');
 		}

 	小于 10K 的 ttf 文件建议 base64 在 css 文件中。例如：

 		@font-face {
   			font-family: "iconfont";
   			src: url('data:application/x-font-ttf;base64,AAEAAAAPAIAAAwBwRkZUTXCKsTYAAAD8AAAAHE9TLzJWulwpAAABGAAAAGBjbWFwzCYhagAAAXgAAAFKY3Z0IAyV/7YAAAmUAAAAJGZwZ20w956VAAAJuAAACZZnYXNwAAAAEAAACYwAAAAIZ2x5Zn46oOwAAALEAAADyGhlYWQHSlJcAAAGjAAAADZoaGVhBzID5wAABsQAAAAkaG10eAq0AM4AAAboAAAAFGxvY2EBjAI0AAAG/AAAAAxtYXhwAScKKwAABwgAAAAgbmFtZQV/3xUAAAcoAAACLnBvc3RMkaPUAAAJWAAAADRwcmVwpbm+ZgAAE1AAAACVAAAAAQAAAADMPaLPAAAAANImhzMAAAAA0iaHMwAEA/QB9AAFAAACmQLMAAAAjwKZAswAAAHrADMBCQAAAgAGAwAAAAAAAAAAAAEQAAAAAAAAAAAAAABQZkVkAMAAeOZFAyz/LABcAxgAHwAAAAEAAAAAAxgAAAAAACAAAQAAAAMAAAADAAAAHAABAAAAAABEAAMAAQAAABwABAAoAAAABgAEAAEAAgB45kX//wAAAHjmRf///4sZvwABAAAAAAAAAAABBgAAAQAAAAAAAAABAgAAAAIAAAAAAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAIAIgAAATICqgADAAcAKUAmAAAAAwIAEErLbBDLLIAADorLbBELLIAATorLbBFLLIBADorLbBGLLIBATorLbBHLLIAADsrLbBILLIAATsrLbBJLLIBADsrLbBKLLIBATsrLbBLLLIAADcrLbBMLLIAATcrLbBNLLIBADcrLbBOLLIBATcrLbBPLLIAADkrLbBQLLIAATkrLbBRLLIBADkrLbBSLLIBATkrLbBTLLIAADwrLbBULLIAATwrLbBVLLIBADwrLbBWLLIBATwrLbBXLLIAADgrLbBYLLIAATgrLbBZLLIBADgrLbBaLLIBATgrLbBbLLAwKy6xJAEUKy2wXCywMCuwNCstsF0ssDArsDUrLbBeLLAAFrAwK7A2Ky2wXyywMSsusSQBFCstsGAssDErsDQrLbBhLLAxK7A1Ky2wYiywMSuwNistsGMssDIrLrEkARQrLbBkLLAyK7A0Ky2wZSywMiuwNSstsGYssDIrsDYrLbBnLLAzKy6xJAEUKy2waCywMyuwNCstsGkssDMrsDUrLbBqLLAzK7A2Ky2waywrsAhlsAMkUHiwARUwLQAAS7gAyFJYsQEBjlm5CAAIAGMgsAEjRCCwAyNwsA5FICBLuAAOUUuwBlNaWLA0G7AoWWBmIIpVWLACJWGwAUVjI2KwAiNEswoJBQQrswoLBQQrsw4PBQQrWbIEKAlFUkSzCg0GBCuxBgFEsSQBiFFYsECIWLEGA0SxJgGIUVi4BACIWLEGAURZWVlZuAH/hbAEjbEFAEQAAAA=') format('truetype');
 		}

* sprite 图片

	Sprite 图片（又称：雪碧图）被运用在众多使用了很多小图标的网站上。sprite 图将众多小图标集成到一张图片上。通过减少请求数提升了性能。在移动端，使用雪碧图时也要注意图片尺寸不能过大，因为图片越大，解码内存消耗就越大，如果过大反而会影响性能。

	解码内存消耗（decoded in memory)的计算公式： （w x h x 4）宽 x 高 x 每个像素4个字节数

	如果设备 DPI 大于1，还需要乘以 DPI 系数，如 Retina 设备 乘以 4 ，RetinaHD 设备乘以 9.

	![](http://gtms02.alicdn.com/tps/i2/TB11_q7HVXXXXbTXVXXWwRTPpXX-1577-730.png)

	考虑到解码内存消耗，合理的生成紧凑的 Sprite 图，既可以带来更少的请求数，又可以保证尽量低的消耗。

* 按需加载


## 渲染性能

### 设备刷新率

设备刷新率是影响我们对于页面滚动、动画流畅性感知的重要参数。

页面运行在设备的浏览器中，现在市面上的移动设备的刷新频率大多是 60次/秒。所以给浏览器渲染每一帧画面的时间应该是（1s/60=16.67ms）。

实际上，浏览器并不是把所有时间都花在页面渲染上，还需要做诸如渲染队列管理、不同线程切换等额外工作。所以单纯的浏览器渲染工作留给我们的时间也就是10ms左右。当每一帧渲染操作的时间大于这个时间时，页面就会表现的比较卡顿。

### 浏览器的页面渲染流程

1. HTML。根据服务器端的 HTML 代码，形成文档对象模型（DOM）
2. Style。 加载并解析样式，形成 CSS 对象模型
3. Layout。在文档对象模型和 CSS 对象模型基础上，生成一颗由一组待生成渲染的对象组成的渲染树;对渲染树上的每个元素，计算其坐标，称之为布局
4. Paint。渲染树上的元素最终展现在浏览器上，称之为“painting”。一般来说，这个绘制过程是在多个层上完成
5. Composite。由上一步可知，对页面中 DOM 元素的绘制时在多个层上进行的。在每个蹭上完成绘制过程之后，浏览器会将所有层按照合理的顺序合并成一个图层，然后显示在屏幕上。

### 如何排查渲染性能问题

首先使用 chrome 的 Timeline ，滚动页面，进行 record 。得到如下结果，其中绿色的波浪线就是页面的帧率。

![](https://img.alicdn.com/tps/TB1JXsbLFXXXXafaXXXXXXXXXXX-712-74.png)

其中波浪线越高表示帧率越高，同时，帧率区域上边标红的一行，表示有问题的帧。

逐一排查标红的帧。

![](https://img.alicdn.com/tps/TB1c2cXLFXXXXbaaXXXXXXXXXXX-531-247.png)

然后，点击选中该帧，可以看到详细的耗时和简单的问题描述。

![](https://img.alicdn.com/tps/TB1jZH6LFXXXXakapXXXXXXXXXX-572-256.png)

本例中，该帧耗时过长，会导致卡顿。

* JS 分析

	发现了运行时间很长的 JS 代码，那么你可以开启 DevTools 中顶部的 JS Profiler 选项：

	![](https://img.alicdn.com/tps/TB1UtwrLFXXXXcXXpXXXXXXXXXX-307-88.jpg)

	开启后，会看到更多细节：

	![](https://img.alicdn.com/tps/TB1Tp.pLFXXXXcOXpXXXXXXXXXX-766-462.png)

	其中，浅绿色的为函数调用的链路，更详细的，可以切换至 Call Tree 的 Tab 页：

	![](https://img.alicdn.com/tps/TB1fyMbLFXXXXX9aXXXXXXXXXXX-760-283.png)

* render 分析

	render 部分包括 Recalculate Style 和 Layout ，如果发现 render 部分耗时过长，需要分别从这两部分进行分析。

	如果某一帧触发了强制 layout ， Timeline 会用红色角标标出:

	![](https://img.alicdn.com/tps/TB1lEEELFXXXXXbXXXXXXXXXXXX-507-174.png)

	如何需要具体分析 Recalculate Style ， 可以选中 Recalculate Style 部分，下面详细列出了受影响的元素个数、触发 Recalculate Style 函数一级警告提示：

	![](https://img.alicdn.com/tps/TB1NCIsLFXXXXbjXpXXXXXXXXXX-853-216.png)

	更加直观的函数调用，可以切换至 Bottom-Up 的 Tab 页，来查看逆向的调用，方便定位触发的函数位置：

	![](https://img.alicdn.com/tps/TB1KBEALFXXXXbsXXXXXXXXXXXX-812-169.png)

	如果需要分析 Layout ，可以选中 Layout 部分，同 Recalculate Style 一样。

* paint 分析

	你可以选中 Timeline 中绿色的 paint 部分：

	![](https://img.alicdn.com/tps/TB1hUMnLFXXXXagXFXXXXXXXXXX-573-574.png)

	summary 会展示绘制的总体情况，包括绘制的元素、元素本身绘制耗时、元素子元素绘制耗时。

	更加详细的信息，可以切换至 Paint Profiler ，包括了每个具体 Paint 的调用和 Paint 区域截图：

	![](https://img.alicdn.com/tps/TB15bUeLFXXXXbEXVXXXXXXXXXX-736-348.png)

* composite 分析

	分析 composite （合成）时，需要选中一帧：

	![](https://img.alicdn.com/tps/TB1sar9LFXXXXcWaXXXXXXXXXXX-680-119.png)

	点击这个 Layers 选项卡，你会看到一个新的视图。在这个视图中，你可以对这一帧中的所有渲染层进行扫描、缩放等操作，同时还能看到每个渲染层被创建的原因：

	![](https://img.alicdn.com/tps/TB1yV7uLFXXXXaMXpXXXXXXXXXX-979-356.png)

	有了这个视图，你就能知道页面中到底有多少个渲染层。如果你在对页面滚动或者渐变效果的性能分析中发现渲染层的合并过程耗费了太多时间（相对于4-5毫秒的预期），那么久可以从这个视图中看到页面中有多少个渲染层，何时被创建，从而对渲染层的数量进行优化。

### Script - 优化JS的执行效率

错误的执行时机和太长的时间消耗是常见的导致 JS 性能低下的原因。

#### 概要

* 对于动画效果的实现，避免使用 setTimeout 或 setInterval ，请使用 requestAnimationFrame。
* 把耗时的 JS 代码放到 Web Workers 中去做。
* 把 DOM 元素的更新划分为多个小任务，分别在多个 frame 中去完成。
* 使用 Chrome DevTools 的 Timeline 和 JS Profiler 来分析 JS 的性能。

#### 使用 requestAnimationFrame

window.requestAnimationFrame 是一个专门为动画而生的 web API 。它通过浏览器在页面重回前执行你的回调函数。通常来说被调用的频率是每秒60次。

很多框架和示例代码都是用 setTimeout 或 setInterval 来实现页面的动画效果，比如 jQuery 中的 animation。这种实现方式的问题是，你在 setTimeout 或者 setInterval 中指定的回调函数的执行时机是无法保证的。他将在这一帧动画的某个时间点被执行，很可能是在帧结束的时候。这就意味着我们可能失去这一帧的信息。

![](https://segmentfault.com/img/bVvdhN)

requestAnimationFrame 的其他高能用法

* 动画： 也是它的主要用途，它将我们动画的执行时机和执行频率交由浏览器决定，以得到更好的性能。
* 函数节流：requestAnimationFrame 的执行频率（一帧）是16.67ms，利用这一特性就可以做到函数节流，避免高频事件在一帧内做多与的无用功的函数执行，例如：

	 	var $box = $('#J_Test'),
	 	$point = $box.find('b');
 		$box.on('mouseenter',function(e){
 		requestAnimationFrame(function(){
 				$point.css({
 	      			top : e.pageY,
	          	left : e.pageX
	  	   		})
    		});
 		});

 * 分帧初始化：同样利用一帧的执行时间将模块的初始化或渲染函数分散到不同的帧中来执行，这样每个模块都有16.67ms的执行时间，而不是一股脑的堆在哪里等着执行。

 		 	var rAF = window.requestAnimationFrame ||  window.webkitRequestAnimationFrame ||
 	     	function(c) {
 	            setTimeout(c, 1 / 60 * 1000);
 	      	};

 		   	function render() {
 	       	self.$container.html(itemHtml);
 	       	self.$container.find('.J_LazyLoad').lazyload();
	     	}

	     	rAF(render);



#### 使用 transform 实现动画 （待定）

css transition 是我们在实现一个动画是能够想到的最简单的一种实现方式。但是当频繁操作样式时，也会出现动画不怎么流畅的问题。

解决这一问题的一个方式就是使用 transform 来实现相同的效果：

	transition: left 2s ease-in-out;  ---> transition: transform 2s ease-in-out;
	left: xxx; ---> transform: translate3d(xxx, yyy, zzz);

#### 硬件（GPU）加速

硬件（GPU）加速，即层的合成（layer composite）。我们常说的使用 translate3D 开启硬件加速，其实是使应用了 translate3D 的元素获得独立的 GraphicsLayer，好处如下：

* 每个 GraphicsLayer 都有一个 GraphicsContext， GraphicsContext 会输出该层的位图，交由 GPU 合成，比 CPU 要快
* 当需要 repaint 时，只要 repaint 自己，不会影响到其他的 GraphicsLayer 。 repaint 完后，只需要通过 GPU 同其它层合并下（composite layers）

所以，在我们平常做动态效果时，会使用 translate3D 的 hack ，利用 GPU 的合成来获取更好的效果和性能。

传统方式下，浏览器依赖于 CPU 来渲染页面内容。而随着 GPU 硬件能力的不断发展，开始试图使用 GPU 的硬件能力来获得更好的性能和更少的电量消耗。使用 GPU 来渲染合成页面内容可以获得明显的速度提升。

##### 合成器

chrome 合成器（Compositor）是一个软件库，用来管理 GraphicsLayer 树。

##### GPU

合成器会使用 GPU 来执行它的合成绘制步骤。在硬件加速体系结构中，合成由 GPU 负责。合成器本质上也是使用 GPU 将层绘制成一个位图，最后输出到屏幕上。



#### 降低代码复杂度或者使用 Web Workers

JS 代码是运行在浏览器的主线程上的。与此同时，浏览器的主线程还负责样式计算、布局，甚至绘制的工作。如果 JS 代码运行时间过长，就会阻塞主线程上其他的渲染工作，很可能就会导致帧丢失。

大多数情况下，可以把纯计算工作放在 Web Workers 中（如果这些计算工作不会涉及 DOM 操作）。一般来说， JS 中数据的处理工作，比如排序或搜索，一般都适合这种处理方式。

	var dataSortWorker = new Worker("sort-worker.js");
	dataSortWorker.postMesssage(dataToSort);

	// The main thread is now free to continue working on other things...

	dataSortWorker.addEventListener('message', function(evt) {
   	var sortedData = e.data;
   	// Update data on screen...
	});

如果 JS 代码需要存取 DOM 元素，也就是说必须在主线程上运行，可以考虑批处理的方式：把任务细分为若干个小任务，每个小任务耗时很少，各自放在一个 requestAminationFrame 中回调运行。

	var taskList = breakBigTaskIntoMicroTasks(monsterTaskList);
	requestAnimationFrame(processTaskList);

	function processTaskList(taskStartTime) {
	  	var taskFinishTime;

  		do {
   	 	// Assume the next task is pushed onto a stack.
   	 	var nextTask = taskList.pop();

   	 	// Process nextTask.
   	 	processTask(nextTask);

   	 	// Go again if there’s enough time to do the next task.
   		 taskFinishTime = window.performance.now();
  		} while (taskFinishTime - taskStartTime < 3);

  		if (taskList.length > 0)
   	 	requestAnimationFrame(processTaskList);

	}

如果采用划分小任务的方式，那么需要确保给用户一个好的 UX/UI，使得用户能够感知到当前浏览器正在处理一个任务，比如一个进度条或者指示器。

#### 避免对 JS 代码进行微优化

对于一个任务，如果换一种实现方式，浏览器的执行速度可以快100倍，这是非常酷的。比如读取一个元素的 offsetTop 属性就比计算它的 getBoundingClientRect() 要快。但一般情况下，在微优化上花再大的精力，整体上 JS 代码的性能也就获得若干毫秒的提升。这是不划算的。

简而言之：慎用微优化。因为一般来说，它对你的 web 应用效果不大。










## 参考资料

1. [无线页面动画优化实例]
2. [WebP]
3. [iconfont]
4. [chrome devtool timeline]
5. [google render performance]


[WebP]: https://developers.google.com/speed/webp/
[iconfont]: http://www.iconfont.cn/
[无线页面动画优化实例]: https://segmentfault.com/a/1190000005017600
[chrome devtool timeline]:https://developer.chrome.com/devtools/docs/timeline
[google render performance]:https://developers.google.com/web/fundamentals/performance/rendering/?hl=zh-cn