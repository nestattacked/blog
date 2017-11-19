---
title: 使用chrome来调试node程序
excerpt: 任何代码都是需要调试的，没有方便的调试工具将会是一件很悲惨的事情。如果我们使用的node版本比较新，就可以很方便地使用chrome进行远程调试。
recommend: 4
tags:
  - node
  - chrome
  - debug
  - 调试
categories:
  - 技术
  - 其他
date: 2017-05-27 00:40:19
thumbnail: debug.png
---
在较新的版本中，node为我们提供了一种新的调试手段：在chrome中远程调试。node和chrome之间通过网络通信来同步debug信息，这让我们能够直接在chrome中调试代码，而这种调试几乎和前端是一样的！

# node端需要做的

为了启动远程调试，我们在执行要调试的程序时，需要添加一些参数来启动debug模式。

```bash
node --inspect=example.com:9229 --debug-brk app.js
```

* `--inspect`参数指定了node程序所在的服务器地址，如果node跑在本地填写localhost就行了，如果跑在云服务器上，就填写云服务器的ip地址或者域名。9229端口是node程序和chrome之间通讯的端口，不需要修改。
* `--debug-brk`参数表示在node程序的第一行打上断点，建议写上让程序停下来，便于我们浏览代码打断点。

# chrome端需要做的

![](console.png)

执行命令之后，控制台会输出一个url。通过访问这个url，chrome就会自动打开调试界面，剩下的调试过程就和前端调试几乎是一模一样的。需要注意的是，node代码执行完毕之后会关闭调试接口，我们需要重复所有这些流程，重新启动一次debug。
