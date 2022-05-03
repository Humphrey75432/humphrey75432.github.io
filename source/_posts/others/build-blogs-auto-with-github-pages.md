---
title: 使用Github Pages来自动部署Hexo博客
date: 2022-05-03 11:40:33
updated: 2022-05-03 11:40:33
tags: 自建博客
categories: 其他闲聊
---

# 一、前言

这个系列的文章网上估计已经烂大街了，但我还是有必要将自己搭建Hexo中遇到的一些坑记录下来。

## 1.1 起因

事情是这样，之前一直有个想法：把自己散落在各大博客网站的文章聚合起来，形成自己的知识体系。所以在二月份就有了自建博客网站的想法。俗话说：“凡事预则立，不立则废”，于是我按照Hexo官网的教程搭建了博客。

早期是使用Travis CI来自动发布博客的，但是呢，Travis CI现在开始收费了，我在免费用了一个月的Travis CI后发现3月23号之后的文章都无法再发布。登上去Travis CI才发现试用结束，要花钱买。挣钱嘛，不寒碜。于是我去看了下Travis CI的订阅价格：

![image-20220503114735833](https://humphrey-blogs-bucket.oss-cn-shenzhen.aliyuncs.com/img/image-20220503114735833.png)

Umm……虽说已经是工作快5年的社畜，身上还是有一点积蓄的。但关键是这玩意儿我不常用啊（谁能保证自己天天都有时间写博客呢），这样下来很不经济。Umm……我猜是新冠疫情让Travis CI嗅到了敏锐的商机，所以才这么堂而皇之的割韭菜吧。Umm。算啦，劳资不用它了。

## 1.2 Travis CI的替代品

Umm，到这里大家可能都会想到Github Actions，这个官方出品的自动化CI/CD工具它不香吗，而且一个月有20000分钟的免费构建时长（说实话这个构建次数对我来说已经完全足够了）。好，那我就用它来构建Hexo博客。

# 二、艰难的配置历程

## 2.1 Jekyll真的垃圾

说实话，要搭建好这一切其实并不容易，我从开始网上寻找文档到真正弄好也花了好几天的时间，期间也踩坑不少。首先需要明白几点：

+ Github Pages采用了Jekyll来生成静态内容，如果使用Hexo来构建博客，必定会死在主题构建这一步（我踏马当时被这个东西折腾的那叫一个难受啊）
+ Jekyll的主题真是太特么丑了，还是Hexo的主题好看一些；
+ 如果是使用github-pages建站，写了CI/CD脚本会交给一个叫"pages-build-deployments"的机器人执行，只有它完成构建，你的博客才能顺理成章地发布；

可想而知当时被Jekyll这个垃圾坑的有多惨了（每次都是在构建主题失败了）

## 2.2 临时解决方案

如果没办法使用自动部署，那么每次发布博客的步骤是这样：

1. 提前按照hexo-deploy-git插件；
2. 每次写完博客在本地执行`hexo clean && hexo deploy`，插件会自动将静态内容发布到gh-pages分支上；
3. 再把本地提交推到master分支；

相当于你要提交两次代码，我觉得完全没必要做重复工作，所以我觉得摸索Github Actions来实现一次提交和部署；

# 三、正确的打开方式

好在办法总比困难多，我最终还是找到了自动化构建的方式，我们去Github Marketplace搜索下面这款插件：`GitHub Pages action`，这款插件已经有了非常丰富的文档。你只需要按照文档的操作步骤来做就行了。

![image-20220503120354326](https://humphrey-blogs-bucket.oss-cn-shenzhen.aliyuncs.com/img/image-20220503120354326.png)

言归正传，我们说一下具体的配置方法：

### （1）创建Github Action的配置文件

使用终端路由到博客的根目录，执行下面的命令：

```shell
$ cd ~/your_hexo_directory
$ mkdir -p .github/workflows
$ cd .github/workflows && touch deploy-hexo.yml
```

YAML文件的名称随意就行

### （2）指定构建分支

```yaml
name: Deploy to gh-pages

on:
  push:
    branches:
      - master
```

### （3）设置构建基础镜像

```yaml
jobs:
  gh-deploy-pages:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
```

### （4）设置构建步骤（在步骤三的基础上设置）

```yaml
steps:
    - uses: actions/checkout@v2

    - name: Setup Node.js
      uses: actions/setup-node@v2.1.0
      with:
        node-version:  '16.x'
      env:
        ACTIONS_ALLOW_UNSECURE_COMMANDS: 'true'

    - name: Cache dependencies
      uses: actions/cache@v2
      with:
        path: ~/.npm
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node-
      env:
        ACTIONS_ALLOW_UNSECURE_COMMANDS: 'true'

    - name: Clean install dependencies
      run: npm install
      env:
        ACTIONS_ALLOW_UNSECURE_COMMANDS: 'true'

   # 换成自己的命令即可，按照package.json里面的设置就行
    - name: Clean and Build app
      run: npm run clean && npm run build
      env:
        ACTIONS_ALLOW_UNSECURE_COMMANDS: 'true'

    - name: deploy to github pages
      uses: peaceiris/actions-gh-pages@v3
      with:
        # 预先生成一个SSH-Key
        deploy_key: ${{ secrets.ACTIONS_DEPLOY_KEY }}
        publish_dir: ./public
      env:
        ACTIONS_ALLOW_UNSECURE_COMMANDS: 'true'
```

生成SSH Keys的命令如下：

```shell
$ cd ~/your_hexo_blog_directory
$ ssh-keygen -t rsa -b 4096 -f gh-pages -N ""
```

然后去Github Project Setting设置如下：

Add Actions Secret，名称为ACTIONS_DEPLOY_KEY，内容为刚才用SSH生成的私钥；

![image-20220503121853836](https://humphrey-blogs-bucket.oss-cn-shenzhen.aliyuncs.com/img/image-20220503121853836.png)

Add Deploy Key，这里记得一定要勾选“Allow Write Access”选项；内容为刚才生成的SSH公钥

![image-20220503122014940](https://humphrey-blogs-bucket.oss-cn-shenzhen.aliyuncs.com/img/image-20220503122014940.png)

### （5）在`gh-pages`目录新增一个`.nojeklly`文件

去Github Repository的gh-pages分支上查看下有没有`.nojeklly`这个文件，如果没有，添加一个。

这个文件的作用是在执行pages-build-deployment不用Jekyll来生成静态内容，这样的话，你的主题用什么就无所谓啦。

![image-20220503122549571](https://humphrey-blogs-bucket.oss-cn-shenzhen.aliyuncs.com/img/image-20220503122549571.png)

### （6）提交配置，去Actions查看效果

![image-20220503122700995](https://humphrey-blogs-bucket.oss-cn-shenzhen.aliyuncs.com/img/image-20220503122700995.png)

可以看到两个构建都成功了，到此大功告成。哈哈！



> 1. https://det171.github.io/blog/automating-building-and-deployment-of-gh-pages/
> 2. https://github.com/marketplace/actions/github-pages-action?version=v3.7.3#%EF%B8%8F-enable-built-in-jekyll-enable_jekyll
