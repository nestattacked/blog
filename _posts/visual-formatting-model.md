---
title: 视觉格式化模型
excerpt: 视觉格式化模型定义了HTML文档的可视化规则，用户代理程序需要按照该模型绘制图像并呈现给用户。
recommend: 5
tags:
  - 前端
  - 视觉格式化模型
  - 绘制
categories:
  - 技术
  - 前端
date: 2018-01-28 16:54:03
thumbnail: cover.jpg
---
# box

HTML文档中的各种元素都会生成对应的box，box就是常说的盒模型中的盒子。通常情况下，一个元素只会生成一个box。比较特殊的，像li这种元素会生成两个box，一个是li标签内部的内容形成的box，另一个是li元素前的点形成的box。

所有box以特定的规则摆放在画布上，最终形成我们在浏览器中看见的图像。

# containing block

## 什么是contaning block

box该以怎么样的规则来摆放呢？实际上，每一个box的摆放位置仅依赖于另一个box，假设boxA依赖于boxB，就可以说boxB是boxA的containing block。containing block这种关系可以理解成box之间的从属关系，属于同一个containing block的box之间需要按照特定的规则来争夺contaning block的空间。

而containing block自身作为一个box也会依赖于它的containing block，containing block这种关系最终会形成树状结构，清晰地告诉我们所有box的从属关系。需要特别说明的是，树的根是一个initial containing block，它没有containing block，它是根元素生成的box，可以理解成整个画布。

## containing block的定义规则

给出一个box，我们该如何找到它的containing block呢？containing block的关系是由box的position属性决定的。

如果position属性是fixed，那么box的containing block就是initial containing block。

如果position属性是absolute，那么box的containing block就是最近的非static祖先生成的box的padding区域（意味着absolute的位置是相对于padding区域的）。

如果position属性是static或者relative，那么box的containing block就是最近的block container祖先生成的box的content区域。block container在后面会详细介绍，display属性为block或者inline-block的元素生成的box都是block container。

# 格式化上下文

第一个格式化上下文是由initial containing block建立的，一个box建立格式化上下文意味着该box下的所有box按照同一套规则摆放，除非子box建立新的上下文。而建立了新的上下文的box和属于新上下文的所有box会作为一个整体参与到它们的上下文。

格式化上下文有两种，分别是BFC和IFC。BFC和IFC代表了两种不同的排版规则。

# block-level box和inline-level box

display属性为block、table、list-item的元素就是block-level box。常见的一些元素，像是h1、p、div默认情况下都是block-level box。一个box如果不是block-level box，那它就是inline-level box。

# 匿名inline-level box

HTML文档中会有一些文本不被任何标签包裹，这种情况下，这些文本会创建一个匿名的inline-level box。下方代码中，span标签会创建一个inline-level box，before和after两段文本会创建两个匿名的inline-level box。就好像他们也是被span标签包裹的一样，不同之处在于它们的样式无法指定，只能从父节点继承或者采用默认值。

```html
<p>before<span>middle</span>after</p>
```

# BFC

## BFC排版规则

建立了BFC的box下的所有box垂直排列，除非子box建立了新的格式化上下文。建立了新的格式化上下文的子box作为整体参与BFC，也遵循垂直排列规则。BFC下的所有box都是block-level box。

## BFC建立条件

根元素生成的box会建立第一个BFC。其他会建立BFC的情况：float元素、绝对定位元素、不是block box的block container、overflow不是visible的block box。为什么overflow不是visible的block box要建立新的BFC？不知道。

## 匿名block-level box

如果BFC中有inline-level box，则会创建匿名block-level box。inline-level box就会属于匿名的block-level box。下方代码中（假设div标签建立了一个BFC），span标签形成的是一个inline-level box，但是BFC下只能存在block-level box。这种情况下，浏览器就会创建一个匿名的block-level box参与到BFC中，而inline-level box属于该block-level box，就好像span标签外还包裹了一层div标签。

```html
<div><span>inline</span></h1>block</h1></div>
```

## margin collapse。

在同一个BFC中，垂直排列的block-level box之间的上下margin会合并。margin都为正时，合并为最大值；都为负时，合并为最小值；正负都有时，合并为相加值。需要注意的是，对于不是建立BFC的box，该box第一个子box的top-margin会传递到该box，box的top-margin就会是自身的top-margin和第一个子box的top-margin的合并值，而在该box内部，第一个子box不会产生top-margin(可以理解成传递给外层box了)。同理，最后一个子box的bottom-margin也是这样的。

# IFC

一个格式化上下文如果建立的不是BFC，那么就是IFC。IFC中的元素都是inline-level box，他们在行中水平摆放，具体的摆放可以参考{% post_link inline-in-block 这篇文章 %}。

# block container box

block cocntainer box是能够建立BFC或者IFC的box。

如果一个box既是block container box又是block-level box，就可以叫做block box。table是一个block-level box但不是block container box。

# float

float元素会尽可能向左或向右移动直到遇上containing block的边缘或者另一个float元素。float元素排斥inline-level box，在同一个BFC下，所有inline-level box都不会与float元素重叠。

## BFC下的float

BFC下，block-level box会照样垂直排序，似乎float元素就是不存在一样。下方代码中，假设div建立了BFC，h1和h2是block-level box，span是一个浮动元素。

```html
<div><h1>block</h1><span>float</span><h2>block</h2></div>
```

![BFC下的float](float_in_bfc.jpg)

如果block-level box建立了新的BFC，float元素会排斥该box，导致block-level box宽度变少。如果上方代码的h2标签建立新的BFC，绘制的图像就会变成下图的样子。这样设计的原因应该是避免float元素排斥inline-level box导致新的BFC排版受到float元素影响，这与BFC的内部排版不受外界影响的设定矛盾。

![float对新BFC的影响](float_in_bfc_with_new_bfc.jpg)

## IFC下的float

IFC下的float元素的表现并没有特殊的地方，大家应该都知道。

# z-index

## 堆叠上下文

首先引入一个概念：堆叠上下文。一个box建立了堆叠上下文意味着：该box下的所有box按照下图规则在z轴上排列。如果子box建立了新的堆叠上下文，其内部的z轴排列顺序单独计算，然后自身作为一个整体在上一级堆叠上下文中排列。

![堆叠上下文中的排列顺序](z-index.jpg)

## 建立堆叠上下文的条件

非static且z-index值不为auto的box会建立一个堆叠上下文。另外，根元素生成的box会建立初始堆叠上下文，也是第一个堆叠上下文呢。
