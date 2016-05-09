---
layout: post
title: 当页面渲染时，浏览器发生了什么
description: 当页面渲染时，浏览器发生了什么
category: fe
---


1. 根据服务器端的 HTML 代码，形成文档对象模型（DOM）
2. 加载并解析样式，形成 CSS 对象模型
3. 在文档对象模型和 CSS 对象模型基础上，生成一颗由一组待生成渲染的对象组成的渲染树
4. 对渲染树上的每个元素，计算其坐标，称之为布局
5. 最后，渲染树上的元素最终展现在浏览器上，称之为“painting”

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

[有关网页渲染，每个前端开发者都该知道的那点事](http://www.html-js.com/article/3000)