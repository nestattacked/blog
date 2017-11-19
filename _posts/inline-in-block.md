---
title: inline元素是如何在block中渲染的
date: 2016-09-25 00:23:54
excerpt: 前端的页面渲染是基于框盒模型的，那么一个inline元素是如何在block中渲染的呢？如果给出一段html和css的代码，能不能在脑海中想象出正确的画面是一种非常重要的能力，能够比较好的反映出我们对于渲染方式的理解是否到位。
recommend: 6
tags:
  - inline
  - 框盒模型
  - 渲染
categories:
  - 技术
  - 前端
thumbnail: inline_in_block_1.jpg
---
# 给出代码

```html
<!DOCTYPE html>
<p>hello<span id="first"></span><span id="second">hello</span></p>
<style>
*{
    padding:0;
    margin:0;
}
p{
    margin-top:100px;
    line-height:40px;
    font-size:20px;
    background-color:red;
}
span{
    width:10px;
    height:10px;
    display:inline-block;
}
#first{
    background-color:blue;
}
#second{
    font-size:20px;
    background-color:grey;
}
</style>
```

上面的代码将会给出怎么样的画面呢？而这样的画面又是根据怎么样的规则来得出的？想不明白吧，想不明白就对了。想明白请你走开。下面我们就一步步地介绍这个过程。

# 从基准线说起

首先，内联元素在排列的时候是依据基准线来分布的。如下所示，就是基准线。基准线就像是我们在练习英文书写时使用的那样，有四条线构成。这四条线分别是text-top、middle、baseline、text-bottom。text-top到text-bottom的高度就是我们书写的范围了，而这个高度是由block容器的font-size决定的。在这里p的font-size被设置为20px，所以text-top到text-bottom的距离就是20px了，middle线正好处于中间的位置，baseline则位于middle和text-bottom之间。

![基准线](inline_in_block_1.jpg)

p中第一个内联的元素是文件hello，这是一个单纯的文字。font-size为20px，浏览器就会把大小为20px的hello字符放在baseline上。需要说明的是，每种字体的baseline可能是不确定的，由字体本身决定。通常情况下，英文字体的baseline就是和书写的baseline一致，为小写字母的下方线。而中文的baseline通常会处于中文下方线稍微偏上一点的位置。放置内容的时候把两者的baseline对齐就可以了。放置hello之后的图如下所示。

![单纯文字](inline_in_block_2.jpg)

第二个内联元素是id为first的span，这个span的display属性被设置成inline-block了。所以会是一个10px*10px的方块，对于没有内部文字的内联元素，他的基准线就是方块的下边线。依据baseline对齐的原则，放置方块之后的图如下所示。

![添加span](inline_in_block_3.jpg)

接下去我们放置第三个内联元素，是一个带有文字hello的sapn标签，同样的，他的display属性被设置成inline-block。我们首先单独考虑这个span的样子，由于宽度和高度已经格式化地确定为10px乘以10px了，这个元素的大小就无法被内部文字撑开。由于内部具有hello这个内联元素，我们同样从基准线开始考虑这个单独的span是如何渲染的。第一步确认基准线的高度，之前已经说过基准线是由font-size决定的，这里的font-size是20px。在得到基于基准线的内容之后，我们就要把整个基准线放到block中去了，line-height这个属性会影响到我们的内联元素的摆放。在这里，line-height并没有显示地指定，所以他会继承父元素的line-height属性，被设置成40px。很显然，我们的基准线只有20px，是要小于line-height的，所以浏览器就会把高度为20px的基准线整个放置到40px的行的中央。最后，流浪器再把这个高度为40px的行放置在10px*10px的block中。所以我们得到一个span内部的结构图。红色框中的内容就是第二个span的最终结果。

![内部的span](inline_in_block_4.jpg)

在得到第二个span之后，浏览器就需要把它放置到p中。在放置这样的内联对象时也需要按照baseline对齐的方式摆放，那么这个span的baseline在哪里呢？这个span的baseline是由内部所有文字的baseline决定的，直接从这些baseline中取最低的就可以了（我猜的）。所以在这里，span框内的hello的baseline就可以代表span的baseline了。放置到p中之后，如下图所示。

![第二个span](inline_in_block_5.jpg)

到了这里以后，我们就得到了p内部的这个基准线组合。最后需要考虑的问题就是如何把整个基准线放到p中了，和之前构造第二个span一样。p的line-height是40px，所有整个基准线正好是居中摆放到p中的。最终的结果是这样的。

![结果](inline_in_block_6.jpg)

# 总结
* 基准线的高度是由font-size决定的
* 基准线的整体在line中是居中的
* 所有内联元素都和基准线对齐，除非vertical-align指定
* 内联元素的基准线由内部文字决定
