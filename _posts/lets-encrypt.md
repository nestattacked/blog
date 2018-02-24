---
title: 使用 let's encrypt 启动https
date: 2017-05-12 00:23:10
excerpt: https能够很好地保护网站通信的安全，防止第三方窃听篡改。而申请一个免费的https证书也已经是很容易的事情了，有条件的都可以给自己网站加个密，绿色的https看着也很爽嘛，：）。
recommend: 5
tags:
  - https
  - 证书
  - certbot
categories:
  - 技术
  - 其他
thumbnail: certbot.svg
---
# 关于let's encrypt

为了让网站启动https，我们需要从一个被浏览器信任的证书机构那里获得一个证书来保证公钥能够安全地交换而不被篡改。早些时候，证书都是需要钱的。而let's encrypt的目的就是提供免费的证书来加速https的普及。比较方便的是，let's encrypt的证书签发是自动化的，通过一个叫做certbot的脚本程序来完成签发。

- [let's encrypt官网](http://www.letsencrypt.org/)
- [certbot官网](https://certbot.eff.org/)

# 启用https

## 安装certbot

certbot官网提供了各种操作系统和web服务器环境下的certbot获取方法，请选择自己对应的环境依据说明完成安装。我是ubuntu加nginx的环境，下面的命令可以在该环境下安装certbot。

```
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-nginx
```

## 申请新的证书

```
certbot certonly --webroot -w /usr/share/nginx/html/home/ -d nestattacked.com -d www.nestattacked.com -w /usr/share/nginx/html/blog/ -d blog.nestattacked.com
```

- certbot是let's encrypt提供的证书申请和管理程序。
- certonly参数表明我们只对证书进行操作，不去自动修改server的配置文件来启动https。所以获得证书之后，我们需要自己去修改nginx的配置文件来启动https。
- --webroot参数表明使用一个叫做webroot的插件来完成证书申请。webroot插件会利用正在运行的web服务来验证我们对域名具有控制权。该插件要求我们给它提供server路径和对应的域名，-w来指定路径，-d指定域名。第一个域名将被用来命名证书。
- 执行命令后，certbot会为我们生成一张证书，该证书包含我们提供的所有域名

## 配置nginx

```
server {
    listen 443;
    server_name nestattacked.com;
    root /usr/share/nginx/html;
    ...
    ssl on;
    ssl_certificate /etc/letsencrypt/live/nestattacked.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/nestattacked.com/privkey.pem;
    ssl_session_timeout 60m;
}
```

在nginx的配置文件中添加一个server，监听在443端口。之后设置好证书和私钥的路径以及ssl会话有效期。ssl_session_timeout参数设置的有效期指的是建立起对称加密之后，这个加密凭证的有效时间。这里我设置成了60分钟。重启nginx使得配置生效。

# 关于certbot的一些细节

## 目录结构

let's encrypt所有的信息都保存在/etc/letsencrypt目录下。比较重要的几个目录是`archive`、`live`、`renewal`。

- `archcive` 保存了所有证书的真实证书文件。子目录名为证书名称。
- `live` 保存了所有证书的超链接，指向保存在archive中的真实证书。
- `renewal` 更新证书时需要的配置文件。

## 创建重复的证书

在某些特殊情况下，我们可能会需要创建重复的证书。重复有两种含义，第一种是证书所包含的域名相同，第二种是证书命名相同。

对于第一种情况，可以在参数中加入`--duplicate`来告诉certbot创建重复证书。certbot会在archive中创建真正的证书，同时在live中创建新的链接指向新的证书。

```
certbot --duplicate certonly --webroot -w /usr/share/nginx/html/blog/ -d nestattacked.com -d www.nestattacked.com
```

或者，使用`--force-renewal`参数达到更新证书的效果，certbot会在archive中创建新的证书，新证书包含同样的域名，之后将live中的链接指向新的证书。

```
certbot --force-renewal certonly --webroot -w /usr/share/nginx/html/blog/ -d nestattacked.com -d www.nestattacked.com
```

对于第二种情况，`--duplicate`可以解决问题。如果希望对原先证书的域名进行扩展，可以加入`--expand`参数。certbot会在archive中创建新的证书，之后将live中的链接指向新的证书。

```
certbot --expand certonly --webroot -w /usr/share/nginx/html/blog/ -d nestattacked.com -d www.nestattacked.com
```

## 证书不可修改的特性

证书是不可修改的，添加新的域名、更新证书等操作本质上都是创建新的证书。

# 后续维护

let's encrypt签发的证书只有90天的有效期，我们需要在证书过期前刷新证书。另外，使用certbot这个脚本还能查看、删除证书。

## 查看证书

```
certbot certificates
```

可以查看本地所有证书，包括域名、有效期、路径等信息。

## 更新证书

```
certbot renew
```

certbot会根据/etc/letsencrypt目录下的配置文件检查所有证书的有效期，如果证书快要过期，certbot就会自动更新证书。实际上，crontab配置了一个定时任务，周期性调用 `certbot renew` 命令来更新证书。

## 删除证书

```
certbot revoke --cert-path /etc/letsencrypt/live/nestattacked.com/cert.pem
```

如果我们担心证书泄露了或者真的不再需要了，可以使用 `revoke` 吊销证书，这样就不会因为证书泄露而导致安全问题了。

```
certbot delete --cert-name nestattacked.com
```

吊销证书之后，证书的相关文件依然保存在文件系统中。我们可以使用 `delete` 来删除遗留在文件系统的相关文件。这样，一个证书就彻底清除了。

# 自动更新阿里云CDN证书

## Hook

certbot在进行renew操作时，会在对应的时间点执行/etc/letsencrypt/pre-hook.d/、/etc/letsencrypt/renew-hook.d/、/etc/letsencrypt/post-hook.d/下的脚本。这些目录下的脚本分别在renew命令执行前、成功执行后，执行后（无论成功与否）调用。需要特别说明的是，脚本的名字只能由数字、字母、-、_构成，并且需要执行权限，否则将不会被执行。

如果我们想在证书更新完成后重启nginx，使得新的证书能够生效，就可以利用renew-hook。在renew-hook.d目录下创建一个脚本，写入`service nginx reload`即可。

## certUpdater

为了certbot完成renew操作之后，能够自动更新阿里云CDN上的证书，我写了一个node程序certUpdater。这样，证书的事情就再也不用关心了，全自动！

脚本代码放在[github](https://github.com/nestattacked/certUpdater)上，使用方式也在README中有说明。
