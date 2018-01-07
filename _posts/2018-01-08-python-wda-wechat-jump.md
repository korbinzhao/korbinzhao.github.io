---
layout: post
title: 基于 python + WebDriverAgent 的“跳一跳”小程序高分教程
description: 2017年12月28日，微信放出了 6.6.1 版本，在微信首页二楼（下拉出现）位置重磅推出了“跳一跳”小程序，瞬间刷爆朋友圈。在大家忙于游戏的时候，有人独辟蹊径基于 python + WebDriverAgent 实现了通过 PC 远程操控手机“跳一跳”小程序小人自动跳动，将分数刷到了令人发指的地步，悄悄占领朋友圈第一。
categories: 
- python
- 猎奇

tags: 
- python
- 猎奇
---

2017年12月28日，微信放出了 6.6.1 版本，在微信首页二楼（下拉出现）位置重磅推出了“跳一跳”小程序，瞬间刷爆朋友圈。

![](https://gw.alicdn.com/tfs/TB1loFblY_I8KJjy1XaXXbsxpXa-631-329.png)
![](https://gw.alicdn.com/tfs/TB1_oRblY_I8KJjy1XaXXbsxpXa-576-510.png)

在大家忙于游戏的时候，有人独辟蹊径基于 python + WebDriverAgent 实现了通过 PC 远程操控手机“跳一跳”小程序小人自动跳动，将分数刷到了令人发指的地步，悄悄占领朋友圈第一。

![](https://github.com/wangshub/wechat_jump_game/raw/master/resource/image/jump.gif)

目前已经有比较火的几篇文章详细讲如何实现上述操作，但是或多或少存在描述不够详细、参数设置只在某些机型上表现较好的问题。这也造成我在根据这些教程实现过程中踩了几个小坑。本文基于 Macbook + iphone 6s plus 来讲一下如何实现上述过程，也将踩过的坑记录下。

## 环境准备

### 安装 python3
下载并点击安装。下载地址：https://www.python.org/downloads/mac-osx/
在终端 terminal 中输入如下命令，查看是否安装 python3 成功。
```
~ python3 -V
Python 3.6.4 
```
### 创建 python3 虚拟环境

文档地址：https://docs.python.org/3/tutorial/venv.html
方法如下：
```
~ python3 -m venv tutorial-env
~ source tutorial-env/bin/activate
(tutorial-env) ➜  ~
```

### 安装 pip
安装 python 包管理工具 pip。
文档地址：https://pip.pypa.io/en/latest/installing/
方法如下：
```
(tutorial-env) ➜  ~ curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
(tutorial-env) ➜  ~ python get-pip.py
```

### 安装 xcode
通过 appstore 安装 



### 安装 WebDriverAgent
xcode，尽量新版。尽量升级Xcode到最新版，保持iPhone的版本大于9.3。

从github上下载代码

```
git clone https://github.com/facebook/WebDriverAgent
```

安装 carthage

 ```
 brew install carthage
 ```

运行初始化脚本

```
./Scripts/bootstrap.sh
```

该脚本会使用Carthage下载所有的依赖，使用npm打包响应的js文件

执行完成后，直接双击打开WebDriverAgent.xcodeproj这个文件。

#### 设置证书
设置证书签名，Team 一栏勾选个人账号即可。
![](https://gw.alicdn.com/tfs/TB1qoOwjyqAXuNjy1XdXXaYcVXa-1400-803.png)
接着在TARGETS里面选中WebDriverAgentRunner，用同样的方法设置好证书
![](https://gw.alicdn.com/tfs/TB1JEOwjyqAXuNjy1XdXXaYcVXa-1400-803.png)
重命名WebDriverAgent的BundleID，避免重名。
![](https://gw.alicdn.com/tfs/TB1O.OwjyqAXuNjy1XdXXaYcVXa-1400-803.png)
接着在TARGETS里面选中WebDriverAgentRunner，用同样的方法重命名。
![](https://gw.alicdn.com/tfs/TB1ZyA3lLDH8KJjy1XcXXcpdXXa-1400-803.png)

#### 运行和测试
Xcode - Product - Scheme 中选择 WebDriverAgentRunner。
![](https://gw.alicdn.com/tfs/TB1WasZlLDH8KJjy1XcXXcpdXXa-1788-974.png)
将 iphone 通过数据线连接到 macbook 上。
在 Xcode - Product - Destination 中选择数据线连接的 iphone 。
![](https://gw.alicdn.com/tfs/TB1..OwjyqAXuNjy1XdXXaYcVXa-914-649.png)

运行 Xcode - Product - Test

![](https://gw.alicdn.com/tfs/TB1jdcDlNrI8KJjy0FpXXb5hVXa-1288-676.png)

#### 端口转发

```
~ brew install libimobiledevice
~ iproxy 8100 8100
```

使用iproxy --help 可以查到更具体的用法。 这时通过访问http://localhost:8100/status确认WDA是否运行成功。

而inspector的地址是http://localhost:8100/inspector， inspector是用来查看UI的图层，方便写测试脚本用的

![](https://gw.alicdn.com/tfs/TB1I.OwjyqAXuNjy1XdXXaYcVXa-1440-766.png)

## 使用 python 控制 iphone 自动跳一跳

获取 python 跳一跳代码

仓库地址：https://github.com/korbinzhao/wechat_jump_game

```
git clone git@github.com:korbinzhao/wechat_jump_game.git
```

安装 facebook-wda
```
(tutorial-env) ➜ ~ pip3 install --pre facebook-wda
```

安装项目依赖

```
 (tutorial-env) ➜  wechat_jump_game git:(master) ✗ pip3 install -r requirements.txt
```

拷贝 ./config/iPhone 目录下对应的设备配置文件，重命名并替换到 ./config.json

![](https://gw.alicdn.com/tfs/TB1hd1wjyqAXuNjy1XdXXaYcVXa-700-342.png)

在手机中打开小程序界面，运行 python 脚本

![](https://gw.alicdn.com/tfs/TB1c332lLDH8KJjy1XcXXcpdXXa-1242-2208.png)

```
(tutorial-env) ➜  wechat_jump_game git:(master) ✗ python3 wechat_jump_auto_iOS.py
```

最终效果

![](https://gw.alicdn.com/tfs/TB1P3GwjyqAXuNjy1XdXXaYcVXa-1242-2208.png)


## 参考资料
1. [教你用 Python 来玩微信跳一跳](https://github.com/wangshub/wechat_jump_game)
2. [微信跳一跳 mac + iphone 图文教程](https://www.jianshu.com/p/ff973a5910ae)
3. [ATX 文档 - iOS 真机如何安装 WebDriverAgent](https://testerhome.com/topics/7220)