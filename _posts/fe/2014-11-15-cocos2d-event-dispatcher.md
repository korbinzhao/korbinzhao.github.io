---
layout: post
title: Cocos2d-X 3.0 事件分发机制
description: 这两天在使用cocos2d-js做一个拔萝卜游戏，研究了一番cocos2d的事件分发机制，总结分享一下。
category: fe
---

##事件

Cocos2d-JS v3.x中事件分发机制进行了重写，事件可以与任意对象绑定，而不是只有Layer才能获取。对象创建自己的事件监听器，然后加入到全局的事件管理器统一管理。

事件监听器有以下几种：

* 触摸事件
* 键盘响应事件
* 鼠标响应事件
* 自定义事件
* 加速计事件

## 事件分发

在了解事件分发机制之前，我们首先要明确什么是事件分发。

对于事件分发，cocos2d官方定义为：当事件发生(例如，用户触摸屏幕，或者敲键盘)，EventDispatcher 会发布(Event objects)事件对象到合适的EventListeners，并调用你的回调。各个Event object包含事件的信息(比如，触摸点所在的坐标)。

我的理解是对于一个事件（比如触摸事件、键盘相应事件等）可以与任意对象绑定，那么当这个事件被用户触发时，此时应该执行哪一个对象的回调函数（比如此时我们在好几个sprite上同时绑定了touch事件，用户此时点击屏幕，用户此时想点击的到底是哪一个sprite），我们就需要作出判断。EventDispatcher做的就是这个事情。

以touch事件（触摸事件）为例，判断用户点击的是哪一个sprite的方法很简单，其实就是在onTouchBegan方法中获取点击点的坐标pos,然后通过cc.rectContainsPoint(target.getBoundingBox(),pos)判断点击的点是否在SushiSprite上。不过此时又会出现另外一个问题，就是如果两个sprite有相互重叠的部分，而此时用户点击的恰恰是重叠部分，那么怎么判断到底点击的是哪一个sprite呢？这里是通过priority（分为两种：SceneGraphPriority和FixedPriority）来解决的，优先级高的sprite优先执行它所对应的事件监听器的回调函数。这样就形成了一个按照优先级高低排列的sprite队列等待依次执行用户触发的事件，而这中间则通过swallowTouches（吞没事件）属性来控制是否继续向优先级低的sprite传递事件。这就形成了一个完整的事件分发机制。

**tip:**

* SceneGraphPriority(显示优先级)：根据屏幕显示的“遮盖”实际情况，进行有序的函数回调。zOrder越大，优先级越大。

* FixedPriority(固定优先级)：依据手动设定的 Priority 值来决定事件相应的优先级，值越小优先级越高

**总结一下：**

1. 通过点击位置进行点击范围判断，来确定执行哪一个sprite的事件监听器

2. 如果该位置存在重叠的sprite绑定了相同的事件，则依据优先级（SceneGraphPriority显示优先级或FixedPriority固定优先级）来顺序执行函数回调

3. 通过设置swallowTouches属性为true，并在onTouchBegan中返回true或者false来决定是否阻止事件的顺序传递。如果onTouchBegan返回true，且swallowTouches为true，则事件被吞没，事件的顺序传递则被阻止。

## 实例

新建一个sprite，并为其添加一个touch事件：

    var SushiSprite = cc.Sprite.extend({
	    onEnter:function () {
	        cc.log("onEnter");
	        this._super();
	    },
	 
	    onExit:function () {
	        cc.log("onExit");
	    }
    });

    addTouchEventListenser:function(){
	    this.touchListener = cc.EventListener.create({
	        event: cc.EventListener.TOUCH_ONE_BY_ONE,
	        // When "swallow touches" is true, then returning 'true' from the onTouchBegan method will "swallow" the touch event, preventing other listeners from using it.
	        swallowTouches: true,
	        //onTouchBegan event callback function                      
	        onTouchBegan: function (touch, event) { 
	            var pos = touch.getLocation();
	            var target = event.getCurrentTarget();  
	            if ( cc.rectContainsPoint(target.getBoundingBox(),pos)) {
	                cc.log("touched")
	                return true;
	            }
	            return false;
	        }
	    cc.eventManager.addListener(this.touchListener,this);
    });


上面的代码：

1. 首先通过使用cc.EventListener.create创建了一个Touch事件监听器touchListener

2. 然后，通过cc.eventManager.addListener注册监听器到事件管理器。cc.EventListener.create扩展出一个用户监听器。

3. event属性，定义这个监听器监听的类型。

4. swallowTouches属性设置是否吃掉事件，事件被吃掉后不会递给下一层监听器。 

5. onTouchBegan方法处理触摸点击按下事件，我们在这里可以获取到触摸点的坐标pos。event.getCurrentTarget()获取当前事件的接受者，并判断当前的是否点击到了SushiSprite。 

6. 在touch事件中，我们还可以添加onTouchMoved/onTouchEnded方法监听touch移动和结束的回调。如果onTouchBegan返回false后onTouchMoved/onTouchEnded不会执行。

tip: 在onTouchBegan方法中获取点击点的坐标pos,然后通过cc.rectContainsPoint(target.getBoundingBox(),pos)判断点击的点是否在SushiSprite上。


## 参考文献
[[1] http://cn.cocos2d-x.org/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/cocos2d-js/3-jumping-into-cocos2d-js/3-6-creating-user-interaction-with-event-manager/zh.md][1]

[[2] http://cn.cocos2d-x.org/article/index?type=wiki&url=/doc/cocos-docs-master/manual/framework/native/wiki/eventdispatcher-mechanism/zh.md][2]



[1]: http://cn.cocos2d-x.org/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/cocos2d-js/3-jumping-into-cocos2d-js/3-6-creating-user-interaction-with-event-manager/zh.md
[2]: http://cn.cocos2d-x.org/article/index?type=wiki&url=/doc/cocos-docs-master/manual/framework/native/wiki/eventdispatcher-mechanism/zh.md
