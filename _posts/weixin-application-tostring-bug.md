---
title: 微信小程序引入lodash不能正常工作
excerpt: 尝试在微信小程序中引入lodash，但是库似乎没有正常工作。当使用cloneDeep的时候，总是返回空对象。
recommend: 7
tags:
  - 微信小程序
  - bug
categories:
  - 技术
  - TroubleShoot
date: 2017-08-19 15:47:02
thumbnail: weixin-logo.png
---
# lodash在小程序中未正常工作

在微信小程序中引入了lodash库，然后调用了`cloneDeep`方法去拷贝一个对象，但是函数却返回了空对象！最后通过调试，发现原因在于小程序并未实现`Function.prototype.toString`的方法。在开发者社区上提问，得到了微信官方的确认，确实存在问题。他们给出了一个解决办法，在app.js中加上代码（一定要加在lodash前面）：

```javascript
Function.prototype.toString = Object.getPrototypeOf(function(){});
```

# 具体分析

`cloneDeep`会调用`baseClone`，而`baseClone`会调用`getTag`方法去获得拷贝目标的类型，从而判断目标能否拷贝。当他发现拷贝对象是对象时（getTag({value:1})返回'[object Object]'），就会调用对应函数创建新对象。

```javascript
getTag({})//[object Object]
getTag(function(){})//[object Function]
getTag(new Map)//[object Map]
```

`getTag`起初就是`Object.prototype.toString`。但是lodash为了使得`getTag`在各种环境下都能正常工作，有可能会对`getTag`函数做一些改造。而是否修改是基于以下条件的：

```javascript
if (DataView && getTag(new DataView(new ArrayBuffer(1))) != dataViewTag || 
  Map && getTag(new Map) != mapTag || 
  Promise && getTag(Promise.resolve()) != promiseTag || 
  Set && getTag(new Set) != setTag || 
  WeakMap && getTag(new WeakMap) != weakMapTag) {
  getTag = newGetTag;
}
```

这些`DataView`、`Map`、`Promise`、`Set`、`WeakMap`都是取的原生函数，而不是polyfill的结果。所以在正常情况下，由于小程序并未定义任何这些函数，结果都是`undefined`。由于条件不成立，`getTag`还是`Object.prototype.toString`。`getTag({value:1})`的结果是`[object Object]`，结果正常。但是不幸的是，在获取这些原生类时，出错了！

在获取原生类时，lodash会调用`baseIsNative`方法来确认是不是原生类。而判断是不是原生的原理就是首先对一个标准的原生函数调用`Function.prototype.toString`获得源码，之后去掉函数源码的头尾得到`[native code]`作为比较的标准，再对需要判断的函数也调用`Function.prototype.toString`，掐掉头尾和之前的标准对比，查看是否一致。由于在小程序中，`Function.prototype.toString`就是`Object.prototype.toString`，lodash得到的比较标准就会是`[object Function]`，而需要判断的函数也会得到`[object Function]`，lodash错误把所有东西都认为是原生的了！`baseIsNative`总是返回`true`。

由于`baseIsNative`的错误，我们得到了`Promise`。所以lodash对`getTag`进行了改造。


```javascript
getTag = function (value) {
  var result = objectToString.call(value);
  //objectToString === Object.prototype.toString

  Ctor = result == objectTag ? value.constructor : undefined;
  //objectTag === '[object Object]'

  ctorString = Ctor ? toSource(Ctor) : undefined;
  //toSource函数调用Function.prototype.toString来获取源码

  if (ctorString) {
    switch (ctorString) {
      case dataViewCtorString:
        return dataViewTag;
      case mapCtorString:
        return mapTag;
      case promiseCtorString:
        return promiseTag;
      case setCtorString:
        return setTag;
      case weakMapCtorString:
        return weakMapTag;
    }
  }
  return result;
}
```

改造了就改造了，正常情况下也是能够正常工作的，那又是哪里出问题了？问题就出在toSource上，现在的toSource返回的不再是函数源码了，而是调用Object.prototype.toString的结果。当value是对象时，ctorString得到错误的结果`[object Function]`，而各种比较的标准也是通过`toSource`得到的，`Promise`由于存在，在调用`toSource`后得到的也是`[object Function]`。最终，getTag({value:1})返回的结果就成了'[object Promise]'。baseClone一看，这怎么拷贝？给你一个空对象好了。


# 总结
* 在app.js第一行加上`Function.prototype.toString = Object.getPrototypeOf(function(){})`就能解决lodash引用错误的问题
* `Function.prototype.toString`未定义，直接继承了`Object.prototype.toString`
* `baseIsNative`出错导致`getTag`被改造
* `getTag`依赖`Function.prototype.toString`，导致结果错误
* `cloneDeep`依赖`getTag`判断目标的类型，对象被认为是函数，所以返回空对象
