---
title: JavaScript中的异步编程
excerpt: 异步编程是JavaScript中非常重要的部分，但是在最早的时候，我们不得不面对回调地狱这样的问题。好在在新的标准和方案的帮助下，异步编程变得更加易于理解和实现了。
recommend: 5
tags:
  - JavaScript
  - 异步编程
  - 前端
categories:
  - 技术
  - 前端
date: 2017-12-23 23:56:44
thumbnail: cover.png
---
# 回调地狱

最早的时候，如果我们要写一些node程序，就会调用大量的异步接口，最终形成层层嵌套的代码结构。而这种结构会导致代码的可读性大大降低。代码几乎难以理解，维护这样的代码是一件很麻烦的事情。

```javascript
getPost(postId, function(post){
  renderView(post, function(view){
    sendView(view, function(profile){
      log(profile);
    });
  });
});
```

# Promise

## 链式结构

为了解决回调地狱带来的问题，出现了Promise的方案。Promise的目标是将代码的嵌套结构变成链式结构。下面的代码就是链式结构的，Promise并不是这样的但是类似。

```javascript
getPost(postId).renderView(post).sendView(view).log(profile);
```

## Promise对象

每一个Promise对象都代表一系列操作，这些操作可以是同步的或者异步的，Promise对象并不关心。重要的是，当这一系列操作完成之后需要将结果告知Promise对象。

Promise对象在构造时接受一个函数（表示一系列操作）并立即调用它。在调用该函数时，Promise对象会给其传递两个函数，依次是`resolve`和`reject`。当一系列操作完成后，就能够通过这两个函数将结果告知Promise对象。如果一切正常则调用`resolve(value)`，如果发生错误则调用`reject(error)`，这里的`value`和`error`是一系列操作的结果。

```javascript
var promiseObject = new Promise(function(resolve, reject){
  getJson(function(result){
    if (result.success) {
      resolve(result.value);
    } else {
      reject(result.error);
    }
  });
});
```

## 状态

Promise对象拥有状态，刚刚创建的时候为`pending`，resolve函数调用后变为`fulfilled`，reject函数调用后变为`rejected`。状态一旦变成`fulfilled`或者`rejected`后就无法改变了。

```javascript
new Promise(function(resolve, reject){
  resolve('good');
  //reject doesn't work because the state can't be changed anymore
  reject(new Error('bad'));
});
```

## then方法

在创建完一个Promise对象后，可以通过`then`方法来指定回调函数，`then`方法接受两个回调函数`onValue`和`onError`，其中`onError`可选。Promise对象会依据自身状态在正确的时机调用对应的回调函数。如果状态是`pending`则保存回调函数，等到Promise的状态改变后再调用这些保存的函数。如果状态不是`pending`则立即调用回调函数（异步立即执行）。

```javascript
function onValue(value) {
  console.log('get value: ' + value);
}

function onError(error) {
  console.log('get error: ' + error);
}

promiseObject.then(onValue, onError);
```

## then的链式调用

现在来看，Promise比起最初的回调形式并没有太多的改变，仅仅只是把作为参数的回调函数从异步方法拿到了then方法中。多个异步操作组合起来就会像是下面这样，其中`getPost`、`renderView`等方法现在返回一个Promise对象。

```javascript
getPost(postId).then(function(post){
  renderView(post).then(function(view){
    sendView(view).then(function(profile){
      log(profile);
    });
  });
});
```

这显然没有任何改变，不要忘了Promise的目标是链式调用。为了达到链式调用的目的，then方法返回的其实是一个新的Promise对象，该对象代表了then方法中的回调函数的执行。新的Promise对象的状态取决于回调函数的执行结果，有三种情况：

1. 回调函数抛出异常error，直接`reject(error)`。
2. 回调函数返回了一个非Promise对象的值value，直接`resolve(value)`。
3. 回调函数返回了一个Promise对象，那么该对象的状态决定了then方法返回的Promise对象的状态。

所以，我们的代码就可以写成下面这样：

```javascript
getPost(postId).then(functin(post){
  return renderView(post);
}).then(function(view){
  return sendView(view);
}).then(function(profile){
  log(profile);
});
```

整理一下代码就变成：

```javascript
function handlePost(post) {
  return renderView(post);
}
function handleView(view) {
  return sendView(view);
}
function handleProfile(profile) {
  log(profile);
}
getPost(postId).then(handlePost).then(handleView).then(handleProfile);
```

是不是一下就清爽多了？我写的一个Promise的[简陋实现](https://github.com/nestattacked/claras-in-timeline/blob/master/crumble/mypromise.js)。

# Generator

## Generator介绍

generator的本质是一个可控的分段执行的函数，generator由generator函数生成。定义函数时在`function`后加一个`*`就可以将其声明为generator函数了，函数执行后返回一个generator对象。调用generator的`next`方法可以让代码执行，直到遇见`yield`或者`return`为止。`next`方法返回的是一个对象，结构类似这样：`{value:1,done:false}`，其中value是`yield`或者`return`后的值，如果遇到的是`yield`则done的值为`false`，如果遇到的是`return`则为`true`。

```javascript
function* getGenerator() {
  yield 1;
  yield 2;
  yield 3;
  return 4;
}
var generator = getGenerator();
generator.next();
generator.next();
```

next方法接受一个参数，这个参数会取代yield表达式的值。

```javascript
function* getGenerator() {
  var number = yield 1;
  console.log(number);
  return 2;
}
var generator = getGenerator();
generator.next();
generator.next(3);//控制台输出3
```

## 结合Promise使用

如果我们利用generator分段执行的特性，就能够把异步代码写得像同步的一样。每当调用异步方法时，就暂停函数的执行，待其完成后再继续执行。

```javascript
function* onRequestPost(postId) {
  var post = yield getPost(postId);
  var view = yield renderView(post);
  var profile = yield sendView(view);
  log(profile);
}
```

当yield出的Promise对象的状态发生变化时就调用next方法让函数继续执行，Promise的结果正好可以作为next方法的参数传递给generator。为了让generator运行起来，还需要一些额外的代码。

```javascript
var task = onRequestPost(postId);
task.next().then(function(post){
  task.next(post).then(function(view){
    task.next(view).then(function(profile){
      log(profile);
    });
  });
});
```

很丑陋，好在有第三方的库能够帮我们自动运行这些generator，这个库就是[co](https://github.com/tj/co)，我自己也实现了一个[简陋的版本](https://github.com/nestattacked/claras-in-timeline/blob/master/crumble/myco.js)。

# async和await

Generator和Promise结合使用的办法被最新的JavaScript标准采纳了，叫做`async`和`await`。所有的内容和上面都是一致的，仅仅只是把`yield`换成`await`、`function*`换成`async function`。

```javascript
async function onRequestPost(postId) {
  var post = await getPost(postId);
  var view = await renderView(post);
  var profile = await sendView(view);
  log(profile);
}
onRequestPost(postId);
```

最终，异步代码写起来和同步代码几乎一摸一样了。大家过上了幸福的生活。
