---
title: Git Rebase 一篇就够
date: 2022-06-05 11:20:13
updated: 2022-06-05 11:20:13
tags: VCS
categories: 其他
---

# 一、介绍

Git rebase和Git merge是一样的功能，都是将两个分支的提交合并为一个分支的操作。但是又有些许不一样。我们按照场景来讲解。

# 二、准备一个git提交例子

```shell
$ cd ~
$ mkdir git-sample && cd git-sample
$ git init
```

我们初始化一个git仓库，此时默认会为我们创建master分支。我们在master分支创建两个提交。

```shell
$ vim 1.txt
# After modfied some content ...
$ git add .
$ git commit -m "master commit c1"

$ vim 1.txt
# After mofigied some content ...
$ git commit -am "master commit c2"
```

完成上述操作后，你的提交看起来会像现在这样：

![image-20220605113110033](https://humphrey-blogs-bucket.oss-cn-shenzhen.aliyuncs.com/img/image-20220605113110033.png)

我们在最新提交创建一个新的分支叫feat/work代表我们的工作分支。

```shell
$ git check out -b feat/work
```

完成之后，当前的git分支会变成下面这样。

![image-20220605113432126](https://humphrey-blogs-bucket.oss-cn-shenzhen.aliyuncs.com/img/image-20220605113432126.png)

# 三、Merge操作

假设你当前在feat/work分支工作，你需要把你的改动同步到master分支上进行发布。如果你保证你的改动和master分支是完全干净的。那么你可以直接使用merge操作。

![image-20220605114148804](https://humphrey-blogs-bucket.oss-cn-shenzhen.aliyuncs.com/img/image-20220605114148804.png)



# 四、rebase

变基操作主要解决的是分支出现分叉的情况。分叉就是当我的master分支和feat/work分支开始各自前进的时候，导致git分支开始变得凌乱。正常情况下git一定会出现分支凌乱的情况。这个时候使用rebase分支可以将分叉分支合并为一个。让你看起来之前一直在master分支上工作一样。

> 需要注意：rebase操作有一定风险，请确保不会对他人的改动造成影响再进行操作

![image-20220605114631551](https://humphrey-blogs-bucket.oss-cn-shenzhen.aliyuncs.com/img/image-20220605114631551.png)

rebase操作如下：

```shell
$ git checkout feat/work # 如果在该分支就不用操作了
$ git rebase master
```

操作结束后，分支会变成下面这样子：

![image-20220605114937507](https://humphrey-blogs-bucket.oss-cn-shenzhen.aliyuncs.com/img/image-20220605114937507.png)

rebase原理如下：

1. 首先会取消feat/work分支上每一个提交，并以PATCH的方式保存在缓存中；
2. 会将原先分支上的每一个提交和master分支进行比较，并根据时间顺序先后生成新的提交；
3. 依次重复上面的动作，直到所有提交都合并到master分支上；

这个时候如果要让master分支和feat/work分支前进到同一个节点，只需要执行merge操作就可以了。

如果有冲突可以先解决冲突，解决完成后，执行下面的命令继续rebase即可：

```shell
$ git rebase --continue
```

推送的时候，可以加上`--force-with-lease`命令进行操作即可。
