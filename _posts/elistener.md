---
title: E-listener
date: 2016-01-15 21:34:31 
excerpt: 写了一个练习听力的小工具，勉强用用，总比没有好不是么。
recommend: 4
tags:
  - qt
  - 英语
  - 工具
categories:
  - 技术
  - 哇！传说！
thumbnail: e-listener.png
---
曾经有个美好的愿望，如果电影可以用来练习英语听力那多好。ok，自己动手做个吧。本着能用就好的原则用Qt做了一个，简单归简单，起码能用不是。:)

![E-Listener](e-listener.png)

# 使用方法

界面从上到下一个五个区域，分别是：视频显示、信息栏、提示区、输入框、操作区。操作区对应的按钮分别是：打开文件、播放、下一题、显示答案、切换难度。那么该如何使用呢？

首先需要准备两个文件，视频文件和字幕文件，将他们放在同一目录下即可。字幕文件的命名方式和视频文件一样，只需要在后缀后面加一个s：

* 视频文件 xx.avi
* 字幕文件 xx.avis

而字幕文件的内容需要遵守以下规则（字幕文件可以用平时用的字幕文件改造）：

```
//字幕开始时间
00:00:01,700
//字幕结束时间
00:00:04,460
//字幕内容
So if a photon is directed through a plane
```

准备好文件之后就可以直接打开视频文件，在输入框中输入未填写的单词，回车确认。至于难度切换是什么东西，自己去试吧。:)

还要说明的一点，有时候字幕文件的时间有偏差，可以在输入框输入下面的命令进行调整：

```
//表示视频播放提前1000ms，而后面则延迟0ms
-d 1000 0
```

# 下载链接

这里只提供window下的可运行程序，其他操作系统请自行编译。由于Qt依赖系统自身的编码器，我们可能需要下载额外的编码程序，windows下可以安装K-lite Codec Pack，Basic版本就可以了。

* [源码](https://github.com/nestattacked/e-listener)
* [win32可运行程序](http://yun.baidu.com/share/link?shareid=2177334546&uk=2474971635)
* [K-lite Codec Pack](http://www.codecguide.com/download_kl.htm)
* [TBBT的字幕](https://github.com/nestattacked/e-listener/tree/master/subtitle/TBBT01)
