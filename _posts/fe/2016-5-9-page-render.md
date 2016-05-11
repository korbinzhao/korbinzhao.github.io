---
layout: post
title: 当页面渲染时，浏览器发生了什么
description: 当页面渲染时，浏览器发生了什么
category: fe
---


## 当页面渲染时，浏览器发生了什么

1. 根据服务器端的 HTML 代码，形成文档对象模型（DOM）
2. 加载并解析样式，形成 CSS 对象模型
3. 在文档对象模型和 CSS 对象模型基础上，生成一颗由一组待生成渲染的对象组成的渲染树
4. 对渲染树上的每个元素，计算其坐标，称之为布局
5. 最后，渲染树上的元素最终展现在浏览器上，称之为“painting”

## 渲染基础

首先了解下 Blink 引擎是如何渲染页面的。

### Nodes 和 DOM 树
在 Blink 引擎中，页面内容是存储为由 Node 对象组成的树状结构，也就是 DOM 树。每一个 HTML element 元素都有一个 Node 对象与之对应，DOM 树的根节点永远都是 Document Node。

### 从 Nodes 到 RenderObjects

DOM 树中的每个 node 节点都有一个对应的 RenderObject 对象。RenderObject 存储在与 DOM 树相应的树形结构中 —— Render Tree。RenderObject 知道如何在屏幕上 paint node 内容，为实现这一操作，RenderObject 会调用其对应的 GraphicsContext 来执行必要的 draw 操作。其中，GraphicsContext 就是负责将像素写入位图（bitmap）中，这些位图最终展示在屏幕中。

PS: draw != paint，draw 是将像素绘制到屏幕上，而 paint 是生成元素呈现的像素。

### 从 RenderObjects 到 RenderLayers

每一个 RenderObject 都直接或间接（通过其父对象）的同一个 RenderLayer 相关联。

RenderLayer 的作用就是保证页面元素以正确的顺序合成（composited），这样才能正确的展现元素的重叠以及半透明元素等等。会有一些情形，为一些特殊的 RenderObjects 创建一个新的 RenderLayer。以下是常见的一定会新建 RenderLayer 的 RnederObject：

* 页面的根节点对应的RenderObject
* 有明确的 CSS 定位属性（relative、absolute 或者 transform）
* 是透明的
* 有 CSS overflow、CSS alpha（遮罩）或者 CSS reflection
* 有 CSS 滤镜（filter）
* canvas 元素对应的 RenderObject
* video 元素对应的 RenderObject

其他的 RenderObjects 与其最近的拥有 RenderLayer 的父级 RenderObject，拥有同一个 RenderLayer。

### 从 RenderLayers 到 GraphicsLayers

为利用合成器（compositer），满足如下条件的 RenderLayers，会被认为是一个独立的合成层。

* 有 3D 或者 perspective transform 的 CSS 属性的层
* video 元素的层
* canvas 元素的层
* flash
* 对 opacity 和 transform 对应了 CSS 动画的层
* 使用了 CSS 滤镜（filters）的层
* 有合成层后代的层
* 同合成层重叠，且在该合成层上面（z-index）渲染的层

如果 RenderLayer 是一个合成层，那么它有属于自己的单独的 GraphicsLayer，否则它和它最近的又有 GraphicsLayer 的父 layer 共用一个 GraphicsLayer。

每一个GraphicsLayer都有一个GraphicsContext，其对应的RenderLayer会paint进GraphicsContext中。合成器（compositor）最终会负责，将由GraphicsContext输出的位图（bitmap）合并成最终屏幕显示的图案。

### 层压缩（Layer Squashing）
所有的规则都会有漏洞。正如上面提到的，GraphicsLayers会消耗内存和其他资源（随着GraphicsLayer树的大小增长，会使CPU执行时间越来越长）。当一些RenderLayer同一个成为独立合成层的RenderLayer重叠时，就会产生大量的额外的合成层（上节合成层产生条件的最后一条），十分消耗资源。

我们把产生合成层的自身因素（比如有3D变换的层）称之为直接因素，将比如：合成层后代，同合成层重叠等条件成为简介因素。为了防止上述所说的“层爆炸”，当很多element覆盖在因直接因素产生的层之上时，浏览器的排版引擎，会将这些element的RenderLayers覆盖在这个直接因素产生的RenderLayer上，同时将他们压缩（squash）成一个“层”。这就防止了由覆盖引起的层爆炸。更多细节请看这里和这里

### 从GraphicsLayers到WebLayers再到CC Layers
chrome通过WebLayer和cc layer（chrome compositor layer）实现GraphicsLayers。

### 小结
总的来说，为rendering服务的有如下四种树形结构：

* DOM Tree，基本的模型
* RenderObject Tree，同DOM树的可见节点是一一对应的。RenderObject知道如何去paint其相对应的DOM节点
* RenderLayer Tree，由RenderLayers组成，这些RenderLayer对应于RenderObject树的RenderObject。这种对应关系是一对多的。
* GraphicsLayer Tree，由GraphicsLayers组成，这些GraphicsLayer对应于RenderLayer树的RenderLayer。这种对应关系是一对多的。

![](http://img1.tbcdn.cn/L1/461/1/b611c034f6ea0ad700a1319e52076c64316f21a0)



## 重绘

当改变不会影响元素在网页中位置的样式，如 background-color, border-color, visibility 等，浏览器只会用新的样式将元素重绘一遍。

## 重排

当改变影响到文本内容或结构，或者元素位置时，就会发生重排。重排由以下事件触发：

* DOM 操作（元素的添加、删除、修改、改变顺序）
* 内容变化，包括表单域内的文本变化
* CSS 属性的计算或改变
* 添加删除样式表
* 更改“类”的属性
* 浏览器窗口的操作（缩放、滚动）
* 伪类激活（:hover 等）

## 浏览器的优化渲染

* 浏览器会尽可能的将重绘、重构限制在改变元素的区域内。比如，固定或绝对定位的元素，重绘或重排只会影响元素本身或其子元素，静态定位元素会触发后续所有元素的重流。
* 当 JS 有连续会造成重绘或重排的几段代码时，浏览器会缓存这些改变，在代码运行完毕后再将这些改变一次性应用。举个栗子：

		var $body = $('body');
		$body.css('padding', '1px'); // reflow, repaint
		$body.css('color', 'red'); // repaint
		$body.css('margin', '2px'); // reflow, repaint
		// only 1 reflow and repaint will actually happen

	以上代码只会触发一次重绘或重排。
	**值得注意的一点，改变元素的属性会触发强制性重排**。如果我们在上面的代码中加入一行访问元素属性的代码，就会引起两次重排。

		var $body = $('body');
		$body.css('padding', '1px');
		$body.css('padding'); // reading a property, a forced  reflow
		$body.css('color', 'red');
		$body.css('margin', '2px');

## 性能优化的实际意见

* 尽量减少 DOM 操作。在进行复杂的操作时，最好使用“孤立”的元素（与 DOM 脱离，仅保存在内存中的元素），操作完成后再加入 DOM 中。
* 改变元素样式时，被操作的元素处于 DOM 渲染树的位置越深越好。
* 尽量只给位置绝对或者固定定位的元素添加动画元素。


## 参考文献

1. [有关网页渲染，每个前端开发者都该知道的那点事](http://www.html-js.com/article/3000)
2. [GPU Accelerated Compositing in Chrome](http://www.chromium.org/developers/design-documents/gpu-accelerated-compositing-in-chrome)