---
layout: post
title: SVG入门
description: 目前在对SVG进行初步学习，现将学习要点和学习过程中遇到的坑进行一些总结。该文章仅适合SVG入门级童鞋，希望和大家共同进步。
categories: 
- 前端开发
tags: 
- 前端开发
---

##SVG

SVG是使用XML描述的矢量文件，全称可缩放矢量图形，用于描述二维矢量图形的一种图形格式。

**位图&矢量图**

位图（BMP、PNG、JPG等）是基于颜色的描述，由一个个像素点组成；矢量图（SVG、AI等）是基于数学的描述，根据几何特征来绘制图形，矢量可以是一个点或一条线，矢量图只能靠软件生成，特点是放大后图像不会失真，和分辨率无关。

![svg](/images/introduction-to-svg/svg.png "svg") 

##SVG示例

simple.svg

    <svg xmlns="http://www.w3.org/2000/svg" width="200" height="200">
    <!--face-->
    <circle cx="100" cy="100" r="90" fill="#39F"/>
    <!--eyes-->
    <circle cx="70" cy="80" r="20" fill="white"/>
    <circle cx="130" cy="80" r="20" fill="white"/>
    <circle cx="65" cy="75" r="10" fill="black"/>
    <circle cx="125" cy="75" r="10" fill="black"/>
    <!--smile-->
    <path d="M 50 140 A 60 60 0 0 0 150 140" stroke="white" stroke-width="3" fill="none" />
    </svg>

##SVG的使用方式

* 浏览器直接打开
* 在HTML中使用img标签引用
* 直接在HTML中使用SVG
* 作为CSS背景

##基本操作API

创建图形，需要使用document.createElementNS()方法
    
    document.createElementNS(ns, tagName)








[Joebon]:    http://joebon.tk  "Joebon"
