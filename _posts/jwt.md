---
title: 什么是JWT
excerpt: JWT是JSON Web Tokens的缩写，是一种用于身份验证的方式。
recommend: 4
tags:
  - Web安全
  - 身份验证
categories:
  - 技术
  - 其他
date: 2018-03-07 17:13:51
thumbnail: cover.jpg
---
# 什么是JWT

当客户端向服务端请求接口时，通常会进行权限验证，请求者只能在权限范围内进行操作。服务端就需要一种手段来识别请求者的身份，身份验证的方式有很多：session、token等等。

常见的token是一段无意义的字符串，服务端接收到token后需要到数据库查询对应的信息，比如账号、有效期、权限等等。如果这个token本身就包含了对应的信息，是不是就可以减少数据库查询的过程了？听起来很不错，但是直接写在token中的身份信息是不具有权威性的，因为服务端无法确认客户端发送的token中的身份信息是否真实。

为了解决这个问题，可以把token加密获得一个签名。伪造者不知道密码就无法伪造token了，而服务端知道密码就可以对发送来的token进行验证，只有一致的签名才能通过验证。

JWT的想法就是这样的。

# 如何生成JWT

## JWT的结构

JWT有三部分组成：header、payload、signature。三部分通过字符`.`连接到一起，最终的字符串就会是：header.padyload.signature。

## header部分

根据加密算法形成一个JSON，把这个JSON进行base64编码就是header部分了。

```json
{
  "alg": "HS256",
  "typ": "jwt"
}
```

## payload部分

payload和header一样，只是JSON内容不同，通常这里会存储账号、有效期、权限类别等信息，切记不要存储敏感信息。其中有些字段是JWT规范定义好了的，使用时需要符合要求。

```json
{
  "account": "nestattacked",
  "admin": true
}
```

## signatuer部分

假如我们选择了SHA256算法，签名的计算方式就会是：

```
HMACSHA256(
  base64UrlEncode(header) + "."  +
  base64UrlEncode(payload),
  secret
)
```

# 如何使用JWT

用户使用账号密码向服务端发起验证，验证通过后，服务端生成JWT返回给客户端。之后每次请求接口时，都需要将这个token带上。

API服务端接受请求后，校验签名的正确性，只有正确的签名才能继续后续的流程，否则直接报错。

# 参考资料

JWT官网提供了一个在线工具用于生成JWT，该工具可以辅助我们debug。同时还提供了各平台的JWT库。

[JWT官网](https://jwt.io/)
