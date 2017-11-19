---
title: Linux下定时任务的配置：crontab
excerpt: crontab是Linux系统下的一个定时任务配置工具，通过该工具可以让操作系统周期性地执行一些事先设定好的脚本。
recommend: 5
tags:
  - Linux命令
  - crontab
  - 定时任务
categories:
  - 技术
  - 其他
date: 2017-11-05 22:52:56
thumbnail: crontab.jpg
---
# Linux下的定时任务

定时执行一些程序的需求总是存在的，例如每隔一段时间就整理日志文件、删除一些临时文件等等。Linux具有一个叫做cron的后台服务，该服务每隔一分钟会检查系统中的定时任务配置文件，根据配置执行相应的脚本。crontab是一个管理配置文件的工具。

# 配置文件

cron服务的配置文件主要存放在`/var/spool/cron`和`/etc/cron.d`这两个目录，前者配置的任务叫做系统级任务，而后者叫做用户级任务。对于用户级任务，我们可以通过`crontab -e`这条命令来修改配置，该命令本质上就是用文本编辑器打开`/var/spool/cron`下的配置文件，当然，我们也可以直接用文本编辑器来修改配置文件。每一个系统用户在`/var/spool/cron`目录下都会有一个配置文件，文件名就是用户名。而`crontab -e`打开的配置文件就是当前用户的。对于系统级任务，我们可以直接使用文本编辑器修改`/etc/cron.d`目录下的配置文件。

# 配置写法

配置文件中每一行都是一个定时任务，具体的格式如下：

```bash
0 0 1 * 1 root /usr/local/bin/node ~/app.js
```

* `0 0 1 * 1`表示执行任务的时间，从左到右分别指的是分钟、小时、日、月、周几。上面的时间表示每周一或者任意月一号的零点零分执行脚本，`*`表示任意数字。
* `root`表示执行该脚本的用户为root。
* `/usr/local/bin/node ~/app.js`表示要执行的脚本，相当于把这条命令输入到shell里执行。

如果是用户级任务，我们就不能写上`root`了，用户级任务就是用那个用户的身份来执行的，所以不需要指定。如果指定了，cron就会认为需要执行的脚本是`root /usr/local/bin/node ~/app.js`，这反而不能正常工作了。

# 例子

1.每分钟执行脚本。

```bash
* * * * * root /usr/local/bin/node ~/app.js
```

2.每个小时开始的时候执行脚本。

```bash
0 * * * * root /usr/local/bin/node ~/app.js
```

3.每月的1号和15号零点零分执行脚本。

```bash
0 0 1,15 * * root /usr/local/bin/node ~/app.js
```

4.每周日凌晨四点零分执行脚本。

```bash
0 4 * * 7 root /usr/local/bin/node ~/app.js
```

5.每天凌晨2点到4点的零分执行脚本。

```bash
0 2-4 * * * root /usr/local/bin/node ~/app.js
```
