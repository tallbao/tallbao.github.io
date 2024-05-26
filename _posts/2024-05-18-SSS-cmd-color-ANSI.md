---
layout: post
author: naiij
title: "[转载] cmd输出彩色字体"
date: 2024-05-16
music-id: 
permalink: /archives/2024-05-18/1
description: "cmd输出彩色字体(win10 cmd控制台支持ANSI转义序列)"
---

以前执行cmd命令会出现`[0m`这种乱码

搜了下是ANSI转义码导致的 

这些字符用于控制终端的文本格式（如颜色、粗体等）。Windows cmd 默认情况下不支持这些转义码



# 装了插件之后能在cmd中通过一些代码实现华丽的效果，比如改变字体颜色

[下载地址](https://github.com/adoxa/ansicon/releases)

下载最新的文件，解压。

假设使用的是64位系统，那么中打开 ".../ansi185-bin/x64" 文件夹（32位选择x86）

文件夹内容如下：
![](https://gitee.com/naiij/blogs_pictures/raw/master/cnblogs/ansicon/p1.png)

然后在控制台中打开这个文件夹

运行下面代码：
```cmd
ansicon.exe -i
ansicon.exe -l
```
就安装完了。

## 使用
相应的转义序列可以改变 cmd 中的文本位置、颜色等。

如C++代码`cout << "\033[32;1m" << "hello world!"` 会输出绿色的hello world!

效果如下：
![](https://gitee.com/naiij/blogs_pictures/raw/master/cnblogs/ansicon/p2.png)

> [本文作者地址](https://www.cnblogs.com/naiij/p/9772584.html)
