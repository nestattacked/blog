---
title: 微信新的自定义分享接口
excerpt: 微信在新版本中废弃了旧的自定义分享接口，引入了新接口updateAppMessageShareData和updateTimelineShareData。
recommend: 6
tags:
  - 微信
categories:
  - 技术
  - 前端
date: 2018-09-18 22:27:25
thumbnail: cover.png
---
# 新的自定义分享接口

6.7.2版本的微信客户端已不再支持旧的自定义分享接口，如果用户的微信版本大于等于6.7.2，原有的代码将无法正常工作。为了解决这个问题，我们需要将微信SDK版本从1.2.0升级到1.4.0，同时在代码中添加新的接口updateAppMessageShareData和updateTimelineShareData。

# 升级代码

首先，1.4.0版本的微信SDK能够工作在任意版本的微信客户端上，所以我们应当直接升级SDK到1.4.0版本。

其次，新接口只存在于新版本中，旧接口也只存在于旧版本中。为了让代码能够在所有版本客户端中正常运行，我们需要在一份代码中同时调用新旧接口。

需要注意的是，下面这种尝试兼容的写法是不行的（可能是调用不存在的接口会导致SDK发生错误）：

```javascript
wx.updateTimelineShareData(shareOption);
wx.updateAppMessageShareData(shareOption);
wx.onMenuShareTimeline(shareOption);
wx.onMenuShareAppMessage(shareOption);
```

正确的写法应该是：

```javascript
if (wx.onMenuShareTimeline) {
  wx.onMenuShareTimeline(shareOption);
  wx.onMenuShareAppMessage(shareOption);
} else {
  wx.updateTimelineShareData(shareOption);
  wx.updateAppMessageShareData(shareOption);
}
```

提醒一下，记得在jsApliList中添加新接口。经过测试，在这里同时添加新旧接口是没有问题的。
