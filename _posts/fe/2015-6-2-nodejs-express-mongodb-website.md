---
layout: post
title: nodejs+mongdb+express建站学习中所踩得坑
description: 最近在慕课网上学习了通过nodejs、mongdb等建站，学习过程中踩了一些坑，现总结分享一下。过程中使用了nodejs、bootstrap、express、mongodb、mongoose等语言、框架或模块。
category: fe
---

##网站开发框架

在网站开发之前，我们首先对网站开发框架进行设计。主要分为后端、前端、本地开发环境三部分。

1. 网站后端由nodejs支撑，同时安装nodejs框架express，数据库选择mongodb，同时选择对mongodb进行快速建模的工具mongoose，后端模板引擎选择jade，日期和时间的格式化选择moment.js，express、mongoose、jade、moment.js都通过npm模块安装包来安装。

2. 前端选择jQuery类库、Bootstrap样式框架，jQuery和Bootstrap都属于前端静态资源，使用Bower来安装它们，Bower也是一个npm安装模块，因此也需要使用npm安装。

3. 本地开发环境用到less的编译、cssmin样式合并、JSHint前后端单元测试的实现等。

![website framework](/images/nodejs-express-mongondb-website/website-framework.png "website framework")

视频教程详见[imooc.com]

##nodejs的安装

开发前需要先安装[nodejs]，通过nodejs官网即可完成nodejs的安装。

##mongodb的安装

mongodb是一个高性能、开源、无模式的文档型数据库（非关系型数据库），是当前NoSql数据库中比较热门的一种。mongodb中的一条记录就是一个文档（document），它的数据结构由键值对组成。mongodb文档和JSON对象很相似。

拓展：NoSql，全称是Not Only Sql，指的是非关系型数据库。下一代数据库主要解决几个要点：非关系型、分布式、开源、水平可拓展。

开发前mongodb需要单独安装。这也是我在不了解情况下踩过的一个坑，mongoose是mongodb的快速建模工具，通过npm安装mongoose并不会同时安装mongodb，且在安装完成mongobd后，在运行node app.

首先安装[mongodb],js本地查看网站效果前，必须前行启动mongodb，并与node连接起来。在项目文件夹imooc下新建data文件夹用于存放mongdb数据。

    cd d:/fe_exercise/imooc
    mkdir data

进入mongodb的安装目录 D:\Program Files\MongoDB 2.6 Standard下的bin文件夹中执行：

    cd d:/Program Files/MongoDB 2.6 Standard/bin
    mongod --dbpath "d:/fe_exercise/imooc/data"

出现以下结果则证明连接成功。

![mongodb-node-link](/images/nodejs-express-mongondb-website/mongodb-node-link.png "mongodb-node-link")

每次本地查看网站前，用命令行启动mongodb，并将mongodb和node连接是一件很麻烦的事情。可以通过一种相对简单方法可以简化这个过程。具体方法如下：

新建一个名为mongodb.cmd的文件，文件内容如下。

    :: 定位到D盘
    d:
    :: 切换到mongodb的数据库目录
    cd Program files\MongoDB 2.6 Standard\data
    :: 删除数据库锁定记录文件
    if exist mongod.lock del mongod.lock missing
    :: 切换到mongodb的bin目录
    cd ..\bin
    :: 配置mongodb的文档存储目录
    mongod --dbpath "D:\fe_exercise\imooc\data"

双击运行，即可完成启动mongodb，连接mongodb和node的过程。

##mongodb可视化管理工具：rockmongo的安装与配置

rockmongo是一个用PHP5写的mongodb可视化管理工具。

**下载rockmongo**

[rockmongo官网]目前无法访问，可选择另一种替代方式：直接访问[rockmongo github地址]，选择一个版本进行下载，本文选用1.1.7版本。

**安装PHP环境**

rockmongo的运行需要PHP环境，本文选用[AppServ 2.5.1]进行PHP、Apache等环境的打包安装，AppServ是PHP网页架站工具组合包，所包含的软件有：Apache、Apache Monitor、MySQL、PHPMyAdmin等。AppServ 2.5.1中PHP版本为PHP 5.2。下载AppServ点击安装，本文选用安装目录为C:/AppServ。

将上文中从github上下载的rockmongo 1.1.7文件夹重命名为rockmongo，放到AppServ安装目录下的www文件夹中：C:/AppServ/www。

**下载mongodb-php拓展**

根据所安装的PHP版本选择相应的mongo-php驱动拓展，本文PHP版本为5.2，因此选择PHP 5.2 VC6 Thread-Safe Mongo extension：[mongo-php-driver]，将下载目录中的php_mongo.dll文件copy到C:/AppServ/php5/ext中。

**修改php.ini文件**

在php.ini文件中添加extension=php_mongo.dll

**重启Apache服务**

重启Apache服务

**修改rockmongo中的文件config..php**

设置以下内容的USERNAME和PASSWORD，将两者均设为root

    $MONGO["servers"][$i]["control_users"]["root"] = "root";//one of control users ["USERNAME"]=PASSWORD, works only if mongo_auth=false

**使用rockmongo登录mongodb**

访问http://localhost/rockmongo,界面如下：

![login](/images/nodejs-express-mongondb-website/login.png "login")

登录后界面如下：

![dbview](/images/nodejs-express-mongondb-website/dbview.png "dbview")

##项目结构的初始化

进入项目目录，运行Node.js command prompt,运行以下代码：

    - imooc/
     > npm install express
     > npm install jade
     > npm install mongoose
     > npm install bower -g
     > npm install bootstrap

通过npm对项目中使用的框架和模块进行安装之后就可以正式开始项目开发了。具体项目源码见[github仓库]。





[imooc.com]: http://www.imooc.com/learn/75 "imooc.com"
[noejs]: https://nodejs.org/ "nodejs"
[mongodb]: https://www.mongodb.org/ "mongodb"
[mongo-php-driver]: http://pan.baidu.com/s/1eQo33kI "mongo-php-driver"
[rockmongo官网]: http://rockmongo.com/downloads "rockmongo"
[rockmongo github地址]: https://github.com/iwind/rockmongo "rockmongo"
[AppServ 2.5.1]: http://pan.baidu.com/s/1c0dInYg "appserv"
[github仓库]: https://github.com/korbinzhao/exercise/tree/master/imooc "imooc github"

