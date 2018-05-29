---
layout: post
title: keynote 中如何展示代码
description: keynote 中高亮展示代码
categories: 
- keynote

tags: 
- keynote
---

1. 安装代码高亮工具
  ```
  brew install highlight
  ```
2. copy代码或者创建需要展示的代码文件

  ```
  # 如果是copy的代码
  # 注意需要指定 --syntax 扩展名
  #            -u 编码，否则中文会乱码
  #            -t 最好将代码中的tab转换成空格，keynote中\t的展示宽度可能会不一致
  pbpaste | highlight --syntax=sh --style=github -k "Fira Code" -K 18 -u "utf-8" -t 4 -O rtf | pbcopy
  
  # 如果是文件中的代码
  highlight --style=github -k "Fira Code" -K 18 -u "utf-8" -t 4 -O rtf <filename> | pbcopy
  
  ```

3. 直接在keynote中粘贴代码

示例：

![](https://gw.alicdn.com/tfs/TB1823kuf1TBuNjy0FjXXajyXXa-1260-428.png)