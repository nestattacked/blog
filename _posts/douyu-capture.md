---
title: 斗鱼人数和弹幕抓取
date: 2016-01-11 19:43:22
excerpt: 知乎上看见有人在说斗鱼弹幕的抓取，自己也试着弄了一下，没什么好结果，：）。
tags:
  - 斗鱼
  - 抓取
categories:
  - 技术
  - 其他
thumbnail: douyu.jpg
---
突然想统计下斗鱼的人数和弹幕，就试着看看能不能抓取到。直接上结果，只找到了弹幕，而人气（这个名字取得好贱）的数据没搞定。

![人气和弹幕](douyu.jpg)

打开chromo的调试，找了一圈都没有发现。应该是flash提供的数据，因为不是很懂flash的接口，本来打算放弃了。稍微查了一下，发现有人直接抓包找到了，真是太机智了。马上用wireshark抓一下，人肉搜索包数据，发现了斗鱼的三种服务器，都是tcp接口。

服务器的通讯协议是tcp，tcp中的内容的格式如下所示，content开头的参数总是type@=xxx

```c
struct tcp{
    //指tcp的字节数，但是不包括length本身，就是tcp结构字节数-4
    int length;
    
    //数据和length一样，不知道这是为什么
    int code;

    //0x000002b2
    int magic;
    
    //真正内容的结构由一堆参数组成，参数的结构都是arv@=value，整个content以0结束
    char content[];
}
```

第一种服务器ip地址是119.90.49.111（还有另外几个相邻ip提供同样的服务）。首先由浏览器发起请求，type@=adposreq，其他参数包括room_id、user_id之类的。然后服务器返回一个包，type@=adposreq，不知道是不是广告服务器，管他呢，反正没有找到我需要的数据，pass。

![adposreq](douyu_type1_1.jpg)

第二种服务器ip地址是115.231.96.18。这个服务器提供了弹幕服务，可以直接socket编程和服务器通讯来获取弹幕。首先浏览器发起请求，type@=loginreq，其他参数有账号密码、room_id（账号密码似乎是随便填的，都有效）。然后服务器返回包，type@=loginres，返回的数据好像都是垃圾并没有任何用途。然后浏览器在发包，type@=joingroup，关键的参数有两个，分别是rid和gid。rid就是房间的id，而gid并不知道怎么获取，我是直接通过抓包得到，短时间内gid并不会变化。到此为止，服务器就会开始推送弹幕数据，而弹幕数据包括送鱼丸的、火箭的、还有其他看不懂的。通过编程是可以直接获取弹幕的，需要注意的是服务器会在一段时间后主动断开连接，浏览器这边需要发送type@=keeplive的包来保持连接（我是直接断开后重新连接）。

![loginreq](douyu_type2_1.jpg)

![loginres](douyu_type2_2.jpg)

![joingroup](douyu_type2_3.jpg)

![chatmessage](douyu_type2_4.jpg)

最后一种服务器提供了人数数据，但是由于有一些类似哈希过得数据不知道怎么得到，所以并没有办法获取到人气。ip是119.90.49.92，登录和弹幕服务器类似，浏览器发送loginreq，服务器返回longinres。人气数据的获取需要浏览器发送一个包，type@=keeplive，然后返回的包中有一个参数uc就是人气值。这个服务器同时还要提供鱼丸那个排行榜的数据。

![loginreq](douyu_type3_1.jpg)

![人气](douyu_type3_2.jpg)

最后试用了下phantomjs去抓人气，可惜不支持flash！再见。

# 参考资料
* [知乎的一个问题](https://www.zhihu.com/question/29027665)
