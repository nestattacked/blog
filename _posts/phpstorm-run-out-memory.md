---
title: phpstorm消耗大量内存导致卡顿的问题
excerpt: 最近一段时间突然发现phpstorm每隔五分钟就会卡顿一次，通过查看任务管理器可以发现内存被消耗完了。在清除了缓存之后，问题就解决了。
recommend: 2
tags:
  - phpstorm
  - 内存溢出
  - 卡顿
categories:
  - 技术
  - TroubleShoot
date: 2017-06-27 00:38:44
thumbnail: phpstorm.png
---
# 卡顿的现象

在使用phpstorm编写代码的时候，每隔一小段时间就会出现卡死的情况，在等待一段时间后才能恢复正常。通过观察任务管理器我们可以发现phpstorm占用的内存在不段上升，直至用完为止，这个应该就是导致卡顿的原因了。

# 解决的办法

在google了一些资料并做了几个尝试之后，我发现可以通过清空缓存的办法来解决这个问题。打开phpstorm并点击 `FIle` > `Invalidate Caches/Restart`，在弹出的对话框中选择 `Invalidate and Restart` 。重启之后问题解决。
