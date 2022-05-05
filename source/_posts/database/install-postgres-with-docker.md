---
title: 使用Docker安装Postgresql
date: 2022-05-05 22:57:39
updated: 2022-05-05 22:57:39
tags: Postgresql
categories: 数据库
---

最近因为工作原因需要学习一下Postgresql，记录一下使用Docker来玩常规软件的骚操作。

# 一、预先准备

你需要准备下列工具：

+ 一台配置还阔以的电脑（牌子无所谓，内存一定要足够你运行Docker）；
+ 已经安装了Docker for Windows或者Docker for Mac；

OK，准备齐全这两样我们就可以开始了。

> 友情提示：如果下载Docker镜像比较慢，提前把国内的镜像加速配置好，教程网上有很多就不浪费时间讲了。



# 二、拉取镜像

去Docker Hub官方拉取postgresql的官方镜像，不指定版本直接使用官方最新版就可以了：

```shell
$ docker pull postgres
```

拉取下来以后可以在Docker Desktop的Dashboard上看到镜像文件。

![image-20220505230353569](https://humphrey-blogs-bucket.oss-cn-shenzhen.aliyuncs.com/img/image-20220505230353569.png)



# 三、构建镜像

打开终端，把下面的命令内容复制到控制台中，然后敲回车执行：

```shell
$ docker run -it --name postgress --restart always \
	-e TZ='Asia/Shanghai' \
	-e POSTGRES_PASSWORD='abc123' \
	-e ALLOW_IP_RANGE=0.0.0.0/0 \
	-v /<你的本机目录>/postgres/data:/var/lib/postgresql \
	-p 5432:5432 \
	-d postgres
```

解释一下各个参数的含义：

+ `--restart always`：表示每次Docker在重启的时候运行该容器；
+ `TZ='Asia/Shanghai'`：设置时区为中国标准时间；
+ `POSTGRES_PASSWORD`：这个不用多说了吧，设置Postgres的登录密码；
+ `ALLOW_IP_RANGE`：设置外部IP可访问，想用可视化数据库工具连接的话加上这个；
+ `-v 本机目录:/var/lib/postgres`：将容器内的改动挂载到本地卷，这样你在Docker容器中的改动都可以完美持久化到宿主机上，下次直接启动Docker服务数据也不会丢；
+ `-p 5432:5432`：将容器内的端口暴露给外部使用，这个无需多讲；

运行成功后，可以在Dashboard看到运行的状况：

![image-20220505231057461](https://humphrey-blogs-bucket.oss-cn-shenzhen.aliyuncs.com/img/image-20220505231057461.png)

![image-20220505231122994](https://humphrey-blogs-bucket.oss-cn-shenzhen.aliyuncs.com/img/image-20220505231122994.png)



# 四、尝试客户端连接

打开手边的数据库连接工具（例如：Navicat或者DBeavor），我们试试连接Docker中的Postgres实例；

![image-20220505231304869](https://humphrey-blogs-bucket.oss-cn-shenzhen.aliyuncs.com/img/image-20220505231304869.png)

配置好以后点击”测试连接“，如果连接成功，那说明配置一切正常。你可以像使用正常数据库那样去使用Postgres的Docker实例了。
