---
layout: post
title: 剪切板粘贴上传图片功能的实现
description: 平时的开发中我们难免要上传一些网页截图，传统的选择文件上传使用起来不方便，这里介绍一种使用js和node实现的剪切板黏贴上传图片功能，这种方式将大大减少我们在上传图片过程中花费的时间。
category: fe
---

平时的开发中我们难免要上传一些网页截图，传统的选择文件上传使用起来不方便，这里介绍一种使用js和node实现的剪切板黏贴上传图片功能。当我们需要上传截图时，只需手动截图后commond/ctrl+v即可完成图片上传。这种方式将大大减少我们在上传图片过程中花费的时间。

要实现剪切板黏贴上传功能，首先我们要先能获取到在剪切板中的图片，这里给大家介绍一个很好用的js插件：[ImageClipboard]。


## ImageClipboard


[ImageClipboard]是一款在chrome、firefox和opera上有效的可以将剪切板中的图片黏贴到网页上的工具。

### 安装

可以使用bower很简单的安装，如果没有安装bower，请先安装bower,安装使用说明见：[bower：客户端库管理工具]。

    bower install image-clipboard
    
### 使用：将剪切板中的图片黏贴到网页中去

    <div id="box"></div>
    <script type="text/javascript" src="ImageClipBoard.min.js"></script>
    <script type="text/javascript">

     var clipboard = new ImageClipboard('#box', function (base64) {
        //do stuff with pasted image
     });

      //onpaste-callback can also be passed as second argument
      //in the constructor above.
     clipboard.onpaste = function (base64) {
        //do stuff with the pasted image
      });

      //you can also pass in single DOM-element instead of 
      //query as the first parameter.

    </script>
    
  运行以上代码后，div#box中会插入一个img标签，src即为当前剪切板中图片。
    
## 剪切板中图片的获取与上传

通过ImageClipboard，我们可以以base64的形式获取到剪切板中的图片，然后将base64数据作为参数通过POST的方式传输到服务器端。

### 浏览器端代码：
    
    this.props.clipboard.onpaste = function (base64) {
      //do stuff with the pasted image
      //console.log(base64)
    
      $.ajax({
        url: 'http://localhost:2929/api/upload-img',
        dataType: 'JSON',
        data: {
          imgData: base64},
        type: 'POST',
        success: function(data) {
          console.log(data);
        }
      });
    
    };

### 服务器端代码

服务器端获取到base64数据，即可将base64数据转为图片存储或者传送到其他服务器。


    export default function uploadImg(req, res) {
  	   
  	  new Promise((resolve, reject) => {

        var fs = require('fs');
        var base64Data = req.body.imgData.replace(/^data:image\/png;base64,/, "");

        fs.writeFile("out.png", base64Data, 'base64', function(err) {
          console.log(err);
        });  
    
      });
    
    }


[Joebon]:    http://joebon.tk  "Joebon"
[ImageClipboard]: https://github.com/jorgenbs/ImageClipboard
[bower：客户端库管理工具]: http://javascript.ruanyifeng.com/tool/bower.html