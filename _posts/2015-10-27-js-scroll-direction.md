---
layout: post
title: 原生JS实现页面滚动方向检测
description: 判断页面滚动方向是一个常见的需求，下面展示一个使用原始方式判断页面滚动方向的解决方式
categories: 
- 前端开发
tags: 
- 前端开发
---

判断页面滚动方向是一个常见的需求，下面展示一个使用原生JS方式判断页面滚动方向的解决方式。

##html

	<script src="http://g.tbcdn.cn/tb/kimi/0.0.233/kimi.js"></script>

	<body style="height: 6000px;">
		<div class="nav" style="position:fixed; top:0;left:0;width: 100%;height: 60px;background-color: pink;"></div>
		
	</body>
	
##js
	  var $nav = $('.nav');
	  var $result = $('.result');
	  
	  //页面滚动监听事件
	  window.onscroll = function(e){
	    $result.html('swipeDown');
	    scrollFunc();
	    if(scrollDirection == 'down'){
	      $nav.css({
	        position: 'absolute'
	      });
	    }
	    else if(scrollDirection == 'up'){
	      $nav.css({
	        position: 'fixed'
	      });
	    }
	  }
	  var scrollAction = {x: 'undefined', y: 'undefined'}, scrollDirection;
	  
	  //判断页面滚动方向
	  function scrollFunc() {
	    if (typeof scrollAction.x == 'undefined') {
	      scrollAction.x = window.pageXOffset;
	      scrollAction.y = window.pageYOffset;
	    }
	    var diffX = scrollAction.x - window.pageXOffset;
	    var diffY = scrollAction.y - window.pageYOffset;
	    if (diffX < 0) {
	    // Scroll right
	      scrollDirection = 'right';
	    } else if (diffX > 0) {
	    // Scroll left
	      scrollDirection = 'left';
	    } else if (diffY < 0) {
	    // Scroll down
	      scrollDirection = 'down';
	    } else if (diffY > 0) {
	    // Scroll up
	      scrollDirection = 'up';
	    } else {
	    // First scroll event
	    }
	    scrollAction.x = window.pageXOffset;
	    scrollAction.y = window.pageYOffset;
	  }
	  
## github地址

	https://github.com/korbinzhao/exercise/blob/master/js-scroll-direction.html