---
title: git
date: 2016-09-24 23:11:32
description: git是一个版本控制工具，用来管理文件的更新历史。这里将会详细介绍git的使用方法和内部运作的方式，让你不再烦恼！希望看懂的人都能感叹一句，git也就那样。我以后要是再尝试写这样的东西我就剁手，把别人说明白简直难。
tags:
- git
- 版本控制
categories:
- 技术
- 其他
---
## 什么是git

想象一下这样的情景：小亮是一个苦逼的毕业党，日日夜夜都忙着写他的宝贝论文。终于，他写完了他第一个版本的论文“论文1”，开心地交给指导老师了。然而指导老师说，这个摘要不行，删掉重写！无奈之下，小亮删除了摘要并重新写了一份，到这里论文就成了“论文2”。就在小亮沾沾自喜的时候，老师打来了电话说道，昨天假酒喝多了，摘要不用改了，那样挺好的。小亮心中真是XXX了，这哪里还记得本来写了什么啊。

不幸的是，小亮没有顺利通过答辩，并被要求启动新的论文项目。这回小亮学乖了，每一次改动文件之前都会备份一次。这样，他的文件夹之中就变成了：“论文1”、“论文2”、“论文3”、“论文4”、“论文5”。随着论文规模的扩大，小亮还需要维护各种各样的文件，包括图片、excel数据、实验报告等等。而每种文件都拥有数个版本。到了最后，小亮疯了，因为他再也不能轻易地找到他想要的文件了。

```
当前的目录
paper_project
└──paper
|	└──paper_version_1
|	└──paper_version_2
|	└──paper_version_3
└──picture
|	└──picture1_version_1
|	└──picture1_version_2
|	└──picture2_version_3
└──exp_report
    └──exp_report1_version_1
    └──exp_report1_version_2
    └──exp_report2_version_3
```

经历很多之后，小亮的答辩又挂了。学校给了他最后一次机会，希望他能好好把握，毕竟这事情说出去也挺丢人的。两年的论文生涯让这位同学更加老道了，他已然摸索出了自己的方法。每当小亮要修改论文的时候，他就完整地复制整个文件夹，并在新的文件夹上进行修改。这样，他就可以轻松地找到任何时候的论文版本。

```
当前的目录
paper_project
└──version1
|	└──paper
└──version2
|	└──paper
|	└──picture1
|	└──picture2
└──version3
    └──paper
    └──picture1
    └──picture2
    └──exp_report1
```

这样的方式确实可以让论文的版本控制变得容易和高效，但是每次复制文件夹都会导致大量的数据冗余，极度浪费存储空间。根据这样的想法，我们就可以设计一个版本控制工具，使得其容易高效，并且没有过度的空间浪费。而git就是我们想要的。

从现在开始，我们通过一个完整的paper_project来介绍git的使用方式和工作方式。

## git init

首先我们需要创建一个新的目录作为我们的项目目录，并且在该目录中使用git init的命令来初始化。

```bash
mkdir paper_project
cd paper_project
git init
```

初始化命令会在项目目录下生成一个名为.git的文件夹，在该文件夹中包含了各种git在运作中需要使用到的数据。具体的数据就会在相关命令使用到的时候再介绍。自此，一个完整的git项目就已经创建好了。

## git add

```bash
printf 'this is paper' > paper
git add paper
```

在项目目录下，我们就可以开始我们的工作了。首先编写一个论文文件paper，并写入内容this is paper。然后在shell中输入命令git add paper，将paper这个文件加入到git的管理中去。那么这个命令运行之后到底发生了什么呢？其实很简单，命令运行之后将会生成一个文件来表示文件对象，文件中的内容就是this is paper以及其他git需要的属性，而这个文件的名字就是由该文件内容计算得出的hash值。文件最后将会被存储在.git/objects目录下，没错，就是git init命令创建的那个文件夹。所以，文件对象的本质就是一个普普通通的而文件罢了，并没有什么特别的地方。完成文件对象的创建之后，git会把这个文件的hash值写入到.git/index文件中去。在大多数教程中说的暂存区或者stage就是指这个index文件。

```
当前目录
paper_project
└──.git
|	└──objects
|	|	└──aeobxnefkdghruzp <-代表paper的文件对象
|	└──index
└──paper
```

下图表示当前git的状态。左侧为工作目录，拥有一个灰色方块，表示文件paper，内容为this is paper。右侧为对象库，拥有一个蓝色方块，表示文件对象paper，内容也为this is paper。最后还有一个index文件指向蓝色方块，表示文件对象被包含在index中，即paper文件已经在暂存区中了。

![git add](blog_git_1.jpg)

## git commit

```bash
git commit -m"a"
```

在完成文件的编辑并且add到index文件中之后，我们就可以通过commit命令生成一个版本。-m参数表示给这一次的版本更新一个注释，方便日后查看。commit对象其实就是对index的一次备份，而且仅仅只是备份指针，而不是文件本身，这样就不会导致大量的数据冗余问题了。在这里还要介绍一下分支名和HEAD的概念，下图中的master就是一个默认的分支，其实就是一个指针指向了当前的commit对象。HEAD代表了当前的分支指向，表示我们正处于的版本。

![git commit](blog_git_2.jpg)

## 添加文件并commit

```bash
printf 'this is pic' > picture
git add picture
git commit -m"b"
```

在提交了第一个版本的项目之后，我们希望能够修改我们的项目构成，添加一个新的文件。在这里，我们添加一个名为picture的文件，内容为this is pic。然后通过add命令把文件添加到index中去，然后再commit形成新的版本节点，需要说明的是，新的版本节点会指向上一个版本节点，从而形成一条commit链条。在这些操作之后，git内部就会变成下图所示的那样。新的commit对象会指向两个文件对象，paper和picture，这些文件都是存储在objects文件夹中的。而master分支会切换到新的commit节点b上，至于HEAD依旧指向master。

![git commit again](blog_git_3.jpg)

## 修改paper文件并commit

```bash
printf 'new paper' > paper
git add paper
git commit -m"c"
```

修改paper文件的内容为new paper，然后add到index文件中并commit生成新版本。在执行git add paper命令的时候，由于paper文件与之前不再相同，所以需要在objects文件夹生成新的文件对象，新的index文件所指向的paper就是这个新生成的对象。旧的对象依旧会完整地保留在objects文件夹中。目前，git的内部状态如下图所示。

![git commit again](blog_git_4.jpg)

## git branch

```bash
git branch
git branch -r
git branch -a
```

git branch命令如果不带有任何参数，那么他就会显示出本地已经存在的分支，并且会在当前分支前标记*。如果带有-r的参数，则会显示所有远程分支。如果带有-a的参数，就会显示出所有分支，包括远程的和本地的。

```bash
git branch new_branch
```

这条命令会创建一个新的分支new_branch，但是目前的分支依旧指向master，并没有改变。创建新的分支之后，git状态见下图。

![git commit again](http://7xj4u9.com1.z0.glb.clouddn.com/blog_git_5.jpg)

```bash
git branch -d new_branch
```

增加了-d参数之后，分支就会被删除。需要注意的是，我们无法删除当前分支，这么做会报错。

## git checkout

```bash
git checkout new_branch
git checkout HEAD~
```

git checkout命令表示从仓库中提取文件。第一条命令后面跟的是一个分支名，将会把HEAD的指向修改，使其指向new_branch，与此同时，index和工作目录的文件就会被new_branch包含的文件替换掉。第二条命令没有具体的分支名，就会导致HEAD脱离，单独指向HEAD~这个commit。checkout HEAD~之后的git状态如下图所示。

![git commit again](http://7xj4u9.com1.z0.glb.clouddn.com/blog_git_6.jpg)

```bash
git checkout HEAD~ paper
```

上面的命令增加了文件名，这样的话，HEAD所指向的内容就不会改变。命令执行之后，只是把HEAD~这个commit的paper文件提取到index和工作目录罢了。并不会像上面的checkout命令那样把HEAD的指向修改。

## git reset

```bash
git reset HEAD~
git reset HEAD~ --soft -- hard
```

git reset命令在执行之后，会让HEAD所指向的分支内容改变，改变之后的分支会指向新的commit。同时，默认情况下，index的内容会被commit中的内容替换掉。第二条命令增加了两个新的参数，采用--soft参数之后，index的内容不会被替换，如果采用了--hard，则工作目录的内容会被commit内容替换掉。用户可以根据自己的需要适当添加。git reset HEAD~之后，下图是git内部的状态。

![git commit again](http://7xj4u9.com1.z0.glb.clouddn.com/blog_git_7.jpg)

## git checkout和reset的特例

```bash
git checkout -- files
git reset -- files
```

第一条命令会把index中的paper取出来代替工作目录中的paper文件。而第二条命令则会把commit中的paper文件取出代替index中的paper文件。这两条命令正好是和add和commit相反地。为了便于区分正常的checkout和reset，所以单独拉出讲。

## git tag

```bash
git tag v1.0
```

git tag命令相当于给commit对象取一个别名。在这里我们创造了一个名字叫做v1.0的tag，用来指代当前HEAD所指向的那个commit。在之后的命令中，我们就可以使用这个别名来代替commit的名字了。

## git merge

```bash
git merge new
```

git merge命令有一个参数为分支名，这个命令会把当前分支指向的commit对象和new分支指向的commit对象合并成一个新的对象。然后，再把分支new指向新的这个commit对象。这样就完成了merge指令。

需要特别说明的是git将如何处理这些merge。当两个分支中有一个是另一个的后代是，merge就会直接把最新的那个commit作为merge的结果，叫做fast-forward。如果两个分支不符合上述条件时，就必须进行真正的merge了。两个分支中同名的文件，如果内容也相同，则在新的commit中保留一份。如果他们的内容不相同则表示文件具有冲突，git会在文件中标识出不同，等待修改之后再次git add，git commit。这样就可以完成merge了。

## git clone

```bash
git clone git@github.com:nestattacked/myblog.git
git clone -b new_branch git@github.com:nestattacked/myblog.git
```

第一条指令表示把后面那个地址的git项目拷贝到当前目录，默认情况下拷贝master分支。如果想要拷贝特定的分支则需要添加-b new_branch，这样就会拷贝new_branch分支了。在完成拷贝之后，拷贝的来源就会成为我们的一个remote，就是我们的远程库。git项目的地址可以有很多种表示方式，在这里采用了git协议。当然，我们也可以拷贝本地的目录，只需要把地址修改成本地项目的目录就可以了。

## git remote

```bash
git remote -v
git remote add origin2 git@github.com:nestattacked/myblog.git
git remote rm origin
```

git remote命令会把所有远程仓库列出，添加-v参数之后会列出更加详细的信息。如果想要添加远程仓库，就可以执行第二条命令。如果想要删除某些远程库，就可以执行第三条命令。

## git fetch

```bash
git fetch master
```

git fetch master表示从远程仓库取回master分支，而取回来的master分支是一个新的分支，命名为origin/master，这样就不会干扰到本地的分支了。在阅读之后再合并这个分支是比较正确的做法。

## git push

```bash
git push origin master:master
git push origin :master
```

第一条命令表示要把本地的master分支推送到origin这个远程仓库的master分支上去。如果远程仓库的分支和本地分支有冲突，那么什么情况下会发生冲突呢？假设有这种情况，开始的时候本地分支和远程分支相同，而之后的时间里，本地分支添加了新的commit，而远程分支也添加了新的commit，这样的情况就是冲突了。冲突情况下，本地git必须先把远程分支fetch到本地，与本地的同名分支合并之后才可以推送到远程仓库去。第二条命令表示要删除远程的master分支，因为本地的分支为空，把空的分支推送到远程仓库就相当于删除了远程分支了。

## git pull

```bash
git pull origin master
```

这条指令就相当于先执行git fetch origin，然后再执行git merge origin/master。

## git log

```bash
git log --pretty=online --graph
```

git log命令用来输出当前分支的commit记录，git会告诉我们最近的commit都有哪些。--pretty=online表示使用简化的信息展示数据，而--graph则表示附带图形的效果，git会给我们绘制一个文字版的关系图。

## github的使用

```bash
ssh-keygen -t ras -C "your-email@xxx.om"
```

github是一个免费的代码托管网站，我们可以在github上创建远程代码库，用来保存我们的git项目。那么该如何使用github呢？注册账号什么的就不说了。完成注册之后，我们就需要在shell中输入上面的指令来创建一对秘钥，这会在~/下生成.ssh文件夹，打开里面的id_ras.pub这个公钥文件，复制里面的内容。然后打开github的账户配置，在左边选择ssh keys，add ssh key，把拷贝来的密钥输进去就OK了。添加了密钥之后，github就能够识别来自你的linux系统的git请求，允许其进行操作。所以在你切换设备之后需要在新的设备上生成秘钥并添加。

```bash
git config --global user.name "your name"
git config --global user.email "your@email.com"
```

执行上述的两条质量配置你的用户名和邮箱，这样的话，commit对象中就可以记录是谁提交了这个commit了。

完成配置之后，我们就可以在github上点击new repository创建一个新的项目了，完成创建之后，github就会告诉我们如果使用了。最方便的做法就是直接在本地执行git clone git@yourgiturl，这样就建立其本地和远程库的关系了。在本地完成项目之后就可以执行git push origin master把代码推送到github上。

## 参考资料

* [git from the inside out](https://maryrosecook.com/blog/post/git-from-the-inside-out)
* [图解git](http://marklodato.github.io/visual-git-guide/index-zh-cn.html)
* [git远程相关](http://www.ruanyifeng.com/blog/2014/06/git_remote.html)
