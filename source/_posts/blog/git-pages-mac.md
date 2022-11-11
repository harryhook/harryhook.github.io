---
title: Hexo + Github 搭建博客（Mac篇）
date: 2020-02-18 23:12:19
tags: 博客搭建
categories: [建站]
---

本文教你如何利用 github pages 快速白嫖一个博客！

<!--more-->

# 前言

当你看到这篇文章时， 已经默认你对github pages有了一定的了解。

# 准备工作

* git
	
	[Git官网下载](https://git-scm.com/) 
	安装完成后查看git是否安装成功
	
	```java
	git --version
	
	git version 2.18.0
	```
	
* Node.js

	[Node.js下载](https://nodejs.org/zh-cn/)
    安装完成后， node -v 检查下是否安装完成
    
	```java 
	node -v
	v11.7.0
	
	npm -v
	6.5.0
	```
    
* github: xxx.github.io (其中xxx是你github的用户名)

	1. 使用邮箱注册github。
	2. 创建的用户名.github.io的仓库，比如说，如果你的github用户名是xxx，那么你就新建xxx.github.io的仓库（必须是你的用户名，其它名称无效）。
	
* 生成ssh key

	1. 为了能在本地操作github仓库， 还需要生成ssh key, **ssh-keygen -t rsa -C "github 邮箱"**。
	2. vi id_rsa.pub, 拿到密钥添加到github账号设置中即可操作本地仓库同步数据到远程。
	
	3. 打开 [GitHub_Settings_Keys](https://github.com/settings/keys) 将ssh key添加进去。
	4. 不确定是否添加成功， 可以在命令行输入 ssh -T git@github.com。
	
	```java
	cd  ~/.ssh
	ssh-keygen -t rsa -C "你的邮箱地址"   // 然后一路回车
	
	vi id_rsa.pub  	// 将id_rsa.pub中的key添加到github中
	
	ssh -T git@github.com   // 测试key是否添加成功
		
	// 出现这串文字提示你添加成功了
	Hi username! You've successfully authenticated, but GitHub does not
	provide shell access.  	```


# Hexo 安装

使用npm命令安装Hexo，输入：

```java
npm install -g hexo-cli 

hexo init blog   // 可以创建一个blog

// 创建一篇博客文章
hexo new first_page

hexo g

hexo s
```
然后在浏览器输入 [localhost:4000](localhost:4000) 即可看到这篇文章

常见的 hexo 命令有以下几个：
```
hexo new 文章名称  // 在

hexo clean  // 清空缓存， 一般不用也行

hexo g  //生成文章

hexo s // 本地预览
```
更多的指令可以见： [Hexo 命令](https://hexo.io/zh-cn/docs/commands.html)

# 博客部署

以上的操作只是在本地预览文章， 要想让我们的文章让更多的人看见， 需要把文章推送到远程仓库， 发布到xxx.github.io

* 打开blog目录下的 _config.yml 文件， 填上github仓库的地址

``` java
deploy:
  type: git
  repo: 
    github: git@github.com:HarryHook/harryhook.github.io.git
  branch: master
```

上面的这个配置就是为了让hexo知道你要把blog部署在哪个位置，很显然，我们部署在我们GitHub的仓库里。

Git部署插件，输入命令：
```java
npm install hexo-deployer-git --save

// 然后部署三连走一波

hexo clean 
hexo g 
hexo d
```

然后在 xxx.github.io 就可以看到你的博客了

# 更换主题

hexo 默认的主题是 landscape， 在 blog 的 themes 下可以看到你使用的主题， 这个主题也是在 _config.yml 进行配置的， 如果你想尝试其他的主题， 可以到 [Themes](https://hexo.io/themes/) 进行下载， 下载到 blog 的 themes 目录下即可。

主题的配置在根目录下的 _config.yml 进行配置。

```java
theme: yelee  // 与你themes的博客主题名称一致即可
```

# Markdown语法

Mac 下推荐使用 MacDown软件 对 md 文件进行编辑， 因为创建的文章都是  .md的， 一些 markdown 的语法可以参考[菜鸟教程](https://www.runoob.com/markdown/md-tutorial.html)， 多摸索多实践。

# 其他个性化配置

* 添加 gitalk 插件

* PV UV 访问次数
