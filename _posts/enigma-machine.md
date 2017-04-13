---
title: Enigma Machine
date: 2015-12-15 11:34:23
description: 简单介绍一下Enigma Machine的工作方式，真的是很简单地介绍一下。
tags:
- enigma
- 加密
categories:
- 技术
- 其他
---
## 历史
Enigma是一种用于加密与解密的机器，主要的使用者是第二次世界大战中的纳粹德国。加密的原理和普通的字母替换类似。比如规定如下的映射方式:

* a->b
* p->c
* l->d
* e->f

那么apple这个单词就会映射到bccdf。只是Enigma的字母映射方式在每一次操作之后都会发生改变，使得加密的方式更加复杂。

![Enigma Machine](full.jpg)

## 结构
Enigma Machine主要有三个部件构成：

* 转子(rotor)
* 反射器(reflector)
* 接线板(plugboard)

![没有接线板的原理图](300px-Enigma-action.png)

每当按钮按下时，电流就会从上图右侧相应位置流入转子。电流在内部线路中流动直到出口，然后点亮出口对应的灯来给出结果。上图中电流从A处进入，从G处流出。一次操作过后，转子旋转，改变了内部结构，导致映射方式改变。电流再次从A流进时，却是从C出流出。

## 转子
转子是Enigma的核心部件，他将电流的位置进行改变。转子两侧的黄点是金属导体，电流可以流通。电流从一侧进入，经过内部的导线，再从另一侧流出。早期的Enigma有三个转子，到了二战后期的时候，德国海军开始采用五个转子的机器。

![转子两侧的接触点](rotors_large.jpg)

转子内部的导线连接两侧的金属，将电路连通。而导线连接的方式就是加密的关键。

![转子内部导线](rwiring05.jpg)

特别注意的是，当转子旋转一周之后会带动左侧的转子转动一格。在转子中有一个钩子(notch)，如果钩子位于正确的位置，在转子转动时就会带动左侧转子。工作方式类似钟表，秒针在转完一周之后会带动分针，分针带动时针。

## 反射器
当电流经过转子从右侧流到左侧之后，就会进去反射器。反射器会把电信号重新流回到转子中，直至最后从右侧转子流出。反射器的链接总是成对的，从A进入从B出来，那么，从B进入的电流一定会从A出来。为什么要设计反射器呢？原因在于，反射器提供了一个很重要的功能：同样的配置下，密码进入Enigma会返回明文。（不要问我为什么）

![反射器](enigma_3_rotors_lufwaffe_8.jpg)

## 接线板
接线板是在前面的基础上增加的部件，就是下图中的前面板，上面连接着多条导线。而接线板的加入才使得Enigma的破解变得几乎不可能。接线板的作用是在电流进出转子时进行转换。使用者可以连接任意两个字母，如果A与B连接在一起，那么当电流从A进入时会被流到转子入口的B处。而从右侧转子B处流出的电流也会被引导到A对应的灯泡。

![接线板](plugboard.jpg)

## 资料

* [转子以及反射器的一些参数](http://www.codesandciphers.org.uk/enigma/rotorspec.htm)
* [详细资料](http://users.telenet.be/d.rijmenants/en/enigmatech.htm#top)
* [python编写的Enigma模拟器](http://py-enigma.readthedocs.org/en/latest/reference.html)
* [我自己编写的基于C++的模拟器](https://github.com/nestattacked/Nlibrary/tree/master/enigma)
* [在线的模拟器](http://enigma.louisedade.co.uk/enigma.html)
* [知乎的一个回答](https://www.zhihu.com/question/28397034/answer/41739506)

## 图片出处

* http://www.cryptomuseum.com/crypto/enigma/d/img/301443/039/full.jpg
* https://upload.wikimedia.org/wikipedia/commons/thumb/6/6c/Enigma-action.svg/300px-Enigma-action.svg.png
* http://www.dgp.toronto.edu/~lockwood/enigma/rotors_large.jpg
* http://ciphermachines.com/pictures/enigma/plugboard.jpg
* http://enigmamuseum.com/replica/rwiring05.jpg
* http://www.minelinks.com/war/assets/enigma_3_rotors_lufwaffe_8.jpg
