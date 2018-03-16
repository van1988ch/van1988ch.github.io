---
layout: post
title: syncwinproject2linux
description: 用于windows,mac下的c++项目自动同步到linux中自动编译的小程序
category: blog
---

## 问题?
在基于c和c++的项目开发,很多人都遇到一个问题,怎么很好的在不带界面的linux系统中写代码.我呆过2个公司,我发现大致有2种方式.

- 通过samba将linux的项目映射到win,mac的目录下面来编辑代码,但是这有一个问题就是需要每次修改代码还需要上去编译,查看编译过程中的警告和错误,然后来回的修改代码编译这样一个循环中直到编译成功.
- 通过ftp等工具直接将代码拖拽到linux中编译,这非常降低效率.
- 直接在linux种用vi来编辑源代码,这种方法直接给我们这些懒人带来了一些屏障.

## 解决问题
针对上面的问题,我主要还是想用轻量级的工具来解决上面的问题.于是基于python编写了一个[syncwinproject2Linux](https://github.com/van1988ch/SyncWinProjectToLinux)的小脚本.这个脚本没有多少代码,非常的轻量,并且可以很好的融合到ide中.这个脚本支持

- 支持自动的同步文件
- 支持自动文件夹创建和同步
- 支持cmake自动编译,将来也可以支持makefile的自动编译
- 在调用脚本的地方可以输出警告和错误信息
- 可以将脚本配置在ide中,可以在ide种显示警告和错误信息
- 可以多套配置,将源码同步到多个服务器种
- 增量同步项目中的文件

## 命令行调用
python syncwinproject2linux config.ini helloworld

- config.ini是配置文件可以指定不同的配置文件上传到不同的服务器中
- helloworld项目的根目录

## 项目中使用的库
- paramiko ssh库
- pickle 序列化

## 最后
有什么问题可以提issue.也可以关注我的微博[van1988ch](https://weibo.com/2296015293/profile)