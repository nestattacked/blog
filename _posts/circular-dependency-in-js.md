---
title: require返回空对象
excerpt: 通过require引入模块时，返回的内容是空对象，这是因为模块间存在循环依赖。
recommend: 5
tags:
  - 前端
  - javascript
categories:
  - 技术
  - TroubleShoot
date: 2018-04-16 22:43:18
thumbnail: cover.jpg
---
# 奇怪的bug

出现了一个奇怪的bug，报错的内容显示模块方法是undefined。经过排查，发现通过require方法引入的模块是一个空对象。查了一些资料后，确定是循环依赖导致的。

# 循环依赖

假设我们有两个文件`a.js`和`b.js`，它们的内容分别是：

```javascript
//a.js
const b = require('b.js');
module.exports = 'a';

//b.js
const a = require('a.js');
module.exports = 'b';
```

如果模块a被引用，系统就会解析`a.js`。由于模块a依赖模块b，又会去解析`b.js`。但是，模块b又依赖模块a，而模块a正在解析中。所以在这种情况下，直接返回一个空对象，因为`a.js`中的代码还没有执行`module.exports = 'a'`，`module.exports`还是默认值，也就是空对象。

所以，如果我们把`a.js`中的两行代码对换位置，结果就会正常。因为模块b在引用模块a的时候，`module.exports = 'a'`已经执行过了。

# 使用madge找到循环依赖

要解决这个问题只需要找到循环依赖并打破它就可以了。但是在一个庞大的项目中找到循环依赖并不容易，好在我们可以借助工具madge。

安装madge

```bash
npm i -g madge
```

查找循环依赖

```bash
madge --circular your-index.js
```

具体的用法可以在[github](https://github.com/pahen/madge)查看。

# 避免循环依赖

回过头思考这个问题，会发现循环依赖都是由不合理的文件组织方式导致的，文件之间耦合过大。良好的文件组织应该形成有层次的依赖。底层的模块是那些纯的模块，它们不依赖任何东西，底层的模块绝对不能依赖上层的模块。
