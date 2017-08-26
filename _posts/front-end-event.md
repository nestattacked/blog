---
title: 前端事件模型
excerpt: 可视化界面的交互通常是基于事件模型的，浏览器也是如此。事件发生，驱动代码运行，从而使状态发生改变（比如界面变化）。
recommend: 6
tags:
  - 前端
  - 事件模型
categories:
  - 技术
  - 前端
date: 2017-08-26 23:37:58
---
# 事件模型概述

所有计算机交互系统都是由事件驱动的，计算机通过INPUT设备知晓外界的指令，然后执行对应的程序。事件指的是IO事件，鼠标点击、键盘操作、触控板操作、触屏操作等等，这些行为的发生就可以叫做事件发生。每一个事件都可以给它定义一个处理程序，一般叫Handler，事件发生则执行对应的Handler。

## 前端事件模型

浏览器的交互也是通过事件来驱动的，只是浏览器中的事件都已经包装地比较抽象了，不再是鼠标左键点击事件，而是某某DOM元素单击事件。而Handler则可以直接用一个定义良好的javascript函数来表示。所以程序员要做的就是为各种可能发生的事件定义好对应的函数。就是要告诉浏览器，如果某元素被单击了，就执行某个函数。

```javascript
document.getElementById('dom-element').onclick = handler;
```

## 捕获和冒泡

在浏览器中，当一个元素被点击时，它的所有父元素都会被认为发生了点击事件，其中document和window也是父元素。既然大家都被认为是点击了，那应该认为哪个点击事件先发生了呢？所以我们引入了捕获和冒泡的概念，现在，事件分为了捕获事件和冒泡事件，在定义handler时需要说明这个handler是属于捕获还是冒泡。

![捕获和冒泡](capture-bubble.jpg)

当点击发生时，首先发生的是window的点击捕获事件，然后是document的点击捕获事件直至被点击元素父元素的点击捕获事件。接着，被点击对象发生点击事件，该对象也叫做目标对象，目标对象不区分捕获和冒泡事件。最后，目标对象父元素发生点击冒泡事件，向上传递直到window发生点击冒泡事件。需要注意的是，有些事件是没有捕获和冒泡的。至此，事件结束。

# 添加事件处理程序

我们有三种方式定义Handler：HTML属性、DOM元素对象属性、addEventListener函数。前两种方式是同一种机制的不同途径，最后一种方式是独立于前两者的机制。

HTML属性方式：

```html
<button onclick="handler()">click</button>
```

DOM元素对象属性方式：

```javascript
document.getElementById('dom-element').onclick = handler;
```

addEventListener方式：

```javascript
document.getElementById('dom-element').addEventListener('click', handler, true);
```

前两种方式只能为同一个元素定义一个Handler，HTML属性方式和DOM元素属性方式共享这个唯一的Handler定义，后来定义的会覆盖之前定义的。权威指南中说这种方式定义的Handler总是优先调用。但是在Chrome上测试后可以看出，首先被调用的是捕获阶段的函数方式定义的Handler，然后才是属性方式定义的Handler。

函数方法可以在同一个元素上定义多个Handler，根据定义顺序调用。但是，同一个Handler只能定义一次，即便你定义了好几次，它也只会执行一次。

# 删除事件处理程序

添加之后，如果我们想要去掉这些Handler肯定也是可以做到的。HTML属性方式和DOM元素对象属性方式是一样的，采用同样的方法去掉就行了。

DOM元素对象属性方式：

```javascript
document.getElementById('dom-element').onclick = null;
```

removeEventListener方式：

```javascript
document.getElementById('dom-element').removeEventListener('click', handler, true);
```

# 事件模型其他方面

## event对象

浏览器在调用Handler时，会传入一个参数`event`，这个参数中包含了事件的各种信息。比如点击事件的`event`就会包括坐标信息。特别说明的是，HTML属性方式添加的Handler是得不到这个`event`对象的，解决的办法是从window对象中取：`window.event`。

## 默认行为的处理

有一些元素会有默认的事件Handler，比如`<a>`标签，默认的处理逻辑就是打开新的页面。如果我们不想这些默认行为发生，该怎么办呢？直接调用`event.preventDefault()`就可以了。但是得不到`event`对象的时候怎么办呢？让Handler返回`false`。

## 传播控制

我们在前面提到过捕获和冒泡的过程，而这个过程其实是可控的。javascript为我们提供了一种控制传播过程的方法：`event.stopPropagation()`。当某个Handler调用了该方法之后，捕获和冒泡过程后面部分的事件就不会发生了。

## target

`event.target`表示真正发生事件的对象，比如在一个鼠标点击事件中，如果用户点击的是某个按钮，那event.target就是这个按钮。即便是在window的点击事件的Handler中，这个event.target也是这个按钮。

## currentTarget

Handler定义在哪个对象上，`event.currentTarget`就是哪个对象。在Handler中，`this === event.currentTarget`。

# 兼容性问题

前面说的都是标准的模型，实际情况并没有这么理想，没错，该死的IE。

## 该死的IE

IE浏览器从IE9开始才支持标准的事件模型，在IE8及以下版本是微软自己定义的模型。下面列出不同的地方：

* 不支持捕获
* 采用`attachEvent`代替`addEventListener`。`domElement.attachEvent('onclick', handler);`
* 采用`detachEvent`代替`removeEventListener`。`domElement.detachEvent('onclick', handler);`
* `attachEvent`可以多次定义同一Handler，也会调用多次。`detachEvent`一次也只删除一次
* `this === window`，不存在`event.currentTarget`
* 使用`srcElement`代替`target`
* 不存在`preventDefault`，使用`event.returnValue = false;`
* 不存在`stopPropagation`，使用`event.cancelBubble = true;`

## polyfill

好在javascript是一个编程语言，我们可以写一些代码来解决兼容性的问题，除了捕获缺失的问题外都可以解决。MDN上给出了一个polyfill方法，但是并没有解决`attachEvent`可以多次添加同一个函数的问题。我写了一个polyfill，去掉了一些特殊事件的处理过程，能够解决基本事件的兼容性问题。把这段代码加到所有javascript代码前面就可以了。

```javascript
(function () {

  var addedHandlers = [];
  function getWrappedHandler (eventName, element, handler, splice) {
    splice = splice || false;
    var counter = 0;
    while (counter < addedHandlers.length) {
      var addedHandler = addedHandlers[counter];
      if(addedHandler.eventName === eventName && addedHandler.element === element && addedHandler.handler === handler) {
        if (splice) {
          addedHandlers.splice(counter, 1);
        }
        return addedHandler.wrappedHandler;
      }
      counter++;
    }
    return null;
  }

  function wrapHandler (handler, element) {
    var wrappedHandler = function(event) {
      event.target = event.srcElement;
      event.currentTarget = element;
      handler.call(element, event);
    }
    return wrappedHandler;
  }

  if (!Event.prototype.preventDefault) {
    Event.prototype.preventDefault = function () {
      this.returnValue = false;
    }
  }

  if (!Event.prototype.stopPropagation) {
    Event.prototype.stopPropagation = function () {
      this.cancleBubble = true;
    }
  }

  if (!Element.prototype.addEventListener) {
    HTMLDocument.prototype.addEventListener = Window.prototype.addEventListener = Element.prototype.addEventListener = function (eventName, handler) {
      if (getWrappedHandler(eventName, this, handler) === null) {
        var wrappedHandler = wrapHandler(handler, this);
        addedHandlers.push({
          eventName: eventName,
          element: this,
          handler: handler,
          wrappedHandler: wrappedHandler
        });
        this.attachEvent('on' + eventName, wrappedHandler);
      }
    }
  }

  if (!Element.prototype.removeEventListener) {
    HTMLDocument.prototype.removeEventListener = Window.prototype.removeEventListener = Element.prototype.removeEventListener = function (eventName, handler) {
      var wrappedHandler = getWrappedHandler(eventName, this, handler, true);
      if (wrappedHandler) {
        this.detachEvent('on' + eventName, wrappedHandler);
      }
    }
  }

})();
```

# 我们如何选择

明白了前端的事件模型之后，我们就该做出选择了，具体项目中该怎么做呢？最好的办法当然是兼容IE9+了！使用标准模型，幸福一生。实在不行就使用jquery之类的库好了。另外，只使用addEventListener方式定义Handler，而不要使用其他两种方式。
