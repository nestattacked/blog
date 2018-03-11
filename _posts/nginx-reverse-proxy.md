---
title: nginx反向代理
excerpt: 利用nginx，可以将请求转发到其他服务，其他服务将结果返回给nginx，再由nginx返回给请求者。这个过程就叫做反向代理。
recommend: 4
tags:
  - nginx
  - 运维
categories:
  - 技术
  - 其他
date: 2018-03-12 14:46:08
thumbnail: cover.jpg
---
# 反向代理

反向代理是相对正向代理来说的，正向代理指的是客户端代理，而反向代理指的是服务端代理。nginx能够很好地完成反向代理的工作，能高效地将请求转发给其他服务（不同端口或者机器），以此实现多种需求：切割业务、负载均衡、隔离服务器。

## 切割业务

所有API服务的域名可能都是统一的，例如`api.example.com`。但是为了便于管理和部署，经常会把API接口拆分到多个服务器上，每个服务器负责特定的一部分接口。这个时候就可以利用nginx的反向代理功能把`api.example.com`这个服务器的请求转发给不同的API服务器。

## 负载均衡

真正处理业务的服务器的吞吐量是有限的，为了应对大流量必须部署足够多的同样的业务服务器。而连接入口服务器和业务服务器的就是反向代理服务器，这样的反向代理服务器就是负载均衡服务器。

## 隔离服务器

计算机领域有很多问题都可以通过添加一个中间层来解决，nginx反向代理服务器就是一个这样的中间层。通过这个中间层，可以将真实的服务器与外界隔离，只有符合条件的请求才能被转发。对符合条件的请求进行不同的定义，就能达到不同的效果。例如，黑名单功能、身份验证等。除此之外，中间层还能提高系统灵活性，帮助我们实现无缝更新等操作。

# nginx配置

## proxy_pass

在nginx中，为了转发请求，只需要在location中添加proxy_pass指令就可以了。

```
location /some/path/ {
  proxy_pass http://example.com/link/;
}

location /some/path/ {
  proxy_pass http://127.0.0.1:8000;
}
```

需要注意的是，如果转发地址是带有uri的，原请求的url中和location匹配的部分会被转发地址的uri替换（例如上面第一个location）。假设原请求是`http://example.com/some/path/resource`，那么转发后的url就会变成`http://example.com/link/resource`。

如果转发地址不带有uri，则只替换域名（例如上面第二个location）。假设原请求是`http://example.com/some/path/resource`，那么转发后的url就会变成`http://127.0.0.1:8000/some/path/resource`。

## fastcgi_pass

如果转发的目标是某些脚本，例如php。就需要将指令换成fastcgi_pass。像php这种脚本，转发地址通常都是unix域，这样能获得更好的性能。具体地址是在php-fpm配置中指定的。

```
location ~ \.php {
  fastcgi_pass unix:/var/run/php-fpm.sock;
}
```

## proxy_set_header

nginx在转发请求的时候，是会修改部分header的（host等），如果希望将header改成原始请求中的内容，可以使用proxy_pass_header指令。

```
location /some/path/ {
  proxy_set_header Host $host;
  proxy_pass http://127.0.0.1:8000;
}
```

## upstream

配置负载均衡的时候，需要使用upstream指令来描述后端服务组，nginx会自动在这些服务器中选择。在location中配置转发地址时，只需要把服务组的名称填上就行了。

```
upstream backend {
  server backend1.example.com;
  server backend2.example.com:8080;
  server unix:/tmp/backend3;
}

server {
  location / {
    proxy_pass http://backend;
  }
}
```

默认情况下的负载均衡规则是轮询，其他规则还有最少连接数、ip哈希。如果要修改负载均衡规则，只需要在描述服务组的时候添加`least_conn`或者`ip_hash`。

```
upstream backend1 {
  least_conn;
  server backend1.example.com;
  server backend2.example.com;
}

upstream backend2 {
  ip_hash;
  server backend1.example.com;
  server backend2.example.com;
}
```

nginx的负载均衡还支持权重、健康检查。具体可以在[这里](http://nginx.org/en/docs/http/load_balancing.html)查看。
