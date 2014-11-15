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




1. 通过添加事件监听器，将精灵以显示优先级 (SceneGraphPriority) 添加到事件分发器。这就是说，当我们点击精灵按钮时，根据屏幕显示的“遮盖”实际情况，进行有序的函数回调（即：如图中黄色按钮首先进入 onTouchBegan 逻辑处理）。

2. 在事件逻辑处理时，根据各种条件处理触摸后的逻辑，如点击范围判断，设置被点击元素为不同的透明度，达到点击效果。

3. 因为设置了 listener1->setSwallowTouches(true); 并且在 onTouchBegan 中做相应的判断，以决定其返回值是 false 还是 true，用来处理触摸事件是否依据显示的顺序关系向后传递。
注意：与 SceneGraphPriority 所不同的是 FixedPriority 将会依据手动设定的 Priority 值来决定事件相应的优先级，值越小优先级越高。




[Joebon]:    http://joebon.tk  "Joebon"
