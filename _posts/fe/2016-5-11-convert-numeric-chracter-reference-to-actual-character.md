---
layout: post
title: 将 NCR 字符转换为真实字符的方法
description: 存在感对于每个人的生活有多么的重要，可能平时并不是太关注，其实他就是生活的全部
category: fe
---

## Numeric Character Reference

看看维基百科的解释：

A numeric character reference (NCR) is a common markup construct used in SGML and other SGML-related markup languages such as HTML and XML. It consists of a short sequence of characters that, in turn, represent a single character from the Universal Charact

NCR是一种常见的标记结构，用于SGML和其他SGML相似的标记语言，如HTML和XML。它由一个短的字符序列组成,代表一个字符（全球的文字字符）。

NCR编码是由一个与号(&)跟着一个井号(#), 然后跟着这个字符的Unicode编码值, 最后跟着一个分号组成的, 如:

    &#dddd;
    &#xhhhh;
    &#name;

其中, dddd是字符编码的十进制表示, 而hhhh是字符的16进制表示.

以 HTML 为例，这三种转义序列都称作 character reference：
前两种是 numeric character reference（NCR），数字取值为目标字符的 Unicode code point；以「&#」开头的后接十进制数字，以「&#x」开头的后接十六进制数字。
后一种是 character entity reference，后接预先定义的 entity 名称，而 entity 声明了自身指代的字符。
从 HTML 4 开始，NCR 以 Unicode 为准，与文档编码无关。

「中国」二字分别是 Unicode 字符 U+4E2D 和 U+56FD，十六进制表示的 code point 数值「4E2D」和「56FD」就是十进制的「20013」和「22269」。所以——

    &#x4e2d;&#x56fd;
    &#20013;&#22269;

——这两种 NCR 写法都会在显示时转换为「中国」二字。

## 如何将 NCR 字符转换成真实字符


方法如下:

    var regex_num_set = /&#(\d+);/g;
    var str = "Here is some text: &#27599;&#26085;&#19968;&#33394;|&#34013;&#30333;~"


    str = str.replace(regex_num_set, function(_, $1) {
      return String.fromCharCode($1);
    });


    document.write('<pre>'+JSON.stringify(str,0,3));

以上例子使用了 [String.prototype.replace()] 和 String.fromCharCode() 方法. 思路为将字符串中的 NCR 字符逐个获取到 "&#"和";"间的 Unicode 字符编码值, 然后利用 String.fromCharCode() 方法, 将 Unicode 编码转为真实字符.


## 参考资料

1. [&#x开头的是什么编码呢。浏览器可以解释它。如&#20013;&#22269;等同与中文"中国"?](https://www.zhihu.com/question/21390312)
2. [Converting numeric character reference to actual character](http://stackoverflow.com/questions/35501026/converting-numeric-character-reference-to-actual-character/35501590#35501590)
3. [String.prototype.replace()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/replace)
4. [[字符编码]Numeric Character Reference和HTML Entities（一）](http://www.cnblogs.com/shishm/archive/2011/11/24/2261996.html)
