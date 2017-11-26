---
title: JavaScript在浏览器中的执行过程
excerpt: JavaScript是如何在浏览器中执行的，各种情况下的执行顺序又是如何的？我做了一些试验，并依据试验结果编出了一些说法，至少能骗过我自己了。：）。
recommend: 5
tags:
  - 前端
  - JavaScript
categories:
  - 技术
  - 前端
date: 2017-11-26 16:51:21
thumbnail: cover.jpeg
---
# 试验一 获取自己

```html
<body>
  <script id="script">
    var scriptNode = document.getElementById('script');
    console.log(scriptNode);
    document.body.removeChild(scriptNode);
    var scriptNode = document.getElementById('script');
    console.log(scriptNode);
  </script>
</body>
```

如果JavaScript代码试图通过DOM的接口获取自身，能够成功么？这时候DOM中是否已经有这个Script节点了。再通过DOM接口删除自身，这时候还能获取么？

结果表明，第一种情况能够获取，第二种情况不能获取。这个结果说明，浏览器解析到script节点的时候，首先会在DOM中创建好这个节点，之后才开始执行script标签中的代码。代码在执行的时候并没有什么特殊的地方，删除就从DOM中删了，这就是为什么第二种情况获取不到的原因。DOM是一个相对独立的东西，JavaScript可以通过接口摆弄它，HTML解析程序也可以摆弄它。：）。

# 试验二 删除自己

```html
<body>
  <script id="script">
    var scriptNode = document.getElementById('script');
    document.body.removeChild(scriptNode);
    console.log('script is still running!');
  </script>
</body>
```

当JavaScript代码在DOM中删除了自己的时候，剩余的代码还会执行么？`script is still running`还会被输出么？

通过测试，我们发现控制台确实是有输出的。所以我们可以猜测，当浏览器解析到script标签的时候，这一整块代码是一起被执行的，无论代码本身做了什么，一旦开始执行，那么所有的代码都会被执行。

# 试验三 动态加载

```html
<body>
  <script>
    var scriptNode = document.createElement('script');
    scriptNode.src = 'external.js';
    scriptNode.id = 'dynamicScript';
    document.body.appendChild(scriptNode);
    document.body.removeChild(scriptNode);
  </script>
  <script>
    //some code take 10 seconds
  </script>
</body>
```

动态加载的脚本会在什么时候被执行？根据观察，所有内联的代码都会被优先同步地执行。当我们动态加载脚本时，浏览器首先会在DOM节点中添加script节点，然后发出HTTP请求获取动态加载的脚本内容，但此时请求状态是pending，只有等到所有内联代码执行完成后才会真正发送请求。这些动态添加的脚本会在下载完成之后马上被执行。需要特别说明的是，就算这些动态节点被删除了，请求依旧会发送并被执行。

# 试验四 异步代码

```html
<body>
  <script>
    setTimeout(function(){
      console.log('async code is running!');
    }, 0);
  </script>
  <script>
    console.log('sync code is running!');
  </script>
</body>
```

我们都知道JavaScript是事件驱动的，系统会维护一个事件队列，不断从中获取事件并处理。那么事件队列的处理是从什么时候开始的呢？

很显然，事件队列的处理要等到所有同步代码都执行完成之后才会开始运转。在此之前，所有事件都会停留在队列中。

# 总结

首先声明，所有结论都是推测，不能百分百确认其正确性。但是，应该差的不多。

浏览器解析HTML文件，遇到HTML和CSS的内容，就构建DOM树和样式树。DOM树是一个相对独立的存在，JavaScript和浏览器解析程序都能够与其交互。如果遇到JavaScript的内容，首先在DOM中创建script节点，然后执行其中的代码。脚本都是成块地执行，除非发生错误，是不会停止的。

对于动态添加的内容，浏览器首先会在DOM中创建节点，然后创建一个pending状态的HTTP请求。等到所有同步代码（内联代码）执行完成后，才真正将这些请求发送。请求创建后，不会因为对应节点被删除而取消，照样会发送并执行。动态内容下载完成后，会立即被执行。

在同步代码执行完成之后，事件队列就会开始被处理。如果有动态加载的内容下载完成了，会被优先处理。等到空闲时，才会来处理事件队列中的东西。
