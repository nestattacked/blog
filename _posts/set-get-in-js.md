---
title: js中的属性读取
date: 2016-10-22 11:00:23
excerpt: 关于js属性读取的一些记录。为属性设置getter和setter之后会怎样影响属性读取的工作方式呢？原型链上的属性读取又是怎样的呢？
thumbnail: /2016/10/22/set-get-in-js/set和get.jpg
tags:
- js
- setter
- getter
- 属性读写
categories:
- 技术
- 前端
---
# 最简单的情况

```javascript
//声明一个对象
var obj = {}

//设置属性
obj.value = 1;

//读取属性
console.log(obj.value);
//1
```

所有人都知道，js中对象的读取是很简单的。只需要简单地使用.或者[]操作符就可以了。

```javascript
//定义父类
var base = {v:1};

//定义子类，这里采用Object.create创建对象
var child = Object.create(base);

//读属性v，大家都知道的，从原型链中寻找，最后会在父类中找到v
console.log(child.v);
//1

//写属性v
child.v = 2;

console.log(child);
//{v:2}

console.log(base);
//{v:1}
```

现在我们考虑原型链属性读取的情况。读属性的运作方式就是沿着原型链往上找属性，子类没有去父类找，直到找到属性或者原型链到头了。

比较特殊的是写属性的情况，写属性并不会覆盖父类的值，只是简单的在子类上添加新的v，并赋值为2。需要特别说明的是，如果父类中的值是一个对象：var base = {v:{age:1}};。而子类企图修改这个对象中的值时：child.v.age = 2;。这个操作会成功地修改父类中的值，请自己想明白。

# 属性设置为get和set

```javascript
//set和get的使用方法
var obj = {
	get v(){
		console.log('get is called');
	},
	set v(value){
		console.log('set is called');
	}
};

obj.v;
//get is called

obj.v = 1;
//set is called
```

set和get的用法就像是函数定义一样，当你定义了一个set和get的属性v之后，属性的读写就会调用对应的函数，而不会像正常情况下一样读写。需要说明的是！get和set是一对的！就是说，当你设置属性为set而没有设置get，会导致这个属性无法读取。同样的道理，如果只设置为get而没有set，就无法写属性了。

```javascript
var obj1 = {
	get v(){},
	v:1,
	set v(v){}
};
var obj2 = {
	get v(){},
	set v(v){},
	v:1
};
```

如果在get和中间插入一个正常的属性v:1，会导致什么情况呢？经过
测试，我猜测情况是这样的。v:1这样的语句会覆盖掉前面一个的get或者set，所以obj1中就剩下set，而obj2中就剩下get，没错v:1是多余的。

# 原型链 + set/get

恩，现在我们要把get/set和原型链都考虑进来，情况似乎变得很复杂了。其实也很简单的。如果是读属性，系统就沿着原型链往上找属性，找到正常属性就返回值，找到get这种的属性就调用函数。而对于写的情况，如果原型链上没有对于属性的set/get这种设置，就直接在子类上添加新属性，如果原型链上有，则调用对于的set函数。

# 总结

* 以上内容完全是自己的猜测，本人声明不负任何责任，：）。
* set/get是一对的
* 读属性直接在原型链上找，找到get就调用
* 写属性写自己的属性，如果原型链上有get则直接调用get
