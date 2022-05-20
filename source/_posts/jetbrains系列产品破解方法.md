
---
title: Jetbrains旗下所有产品激活方法
date: 2022-04-26 16:59:33
tags:
- idea
- 工具
categories:
- [技术,工具]
toc: true
description: 在网络注册码乱象丛生的今天，我们再难以像以前那样找到稳定可用的方法了
---

## 前言  

> 在网络注册码乱象丛生的今天，我们再难以像以前那样找到稳定可用的方法了。

> 随着 [知了大佬](https://zhile.io/) 的出现，我们终于看到了白嫖的曙光————无限试用插件[`IDE Eval Reset`](https://zhile.io/2020/11/18/jetbrains-eval-reset-deprecated.html)。

> 然而，Jetbrains道高一尺，随着idea`2021.2.3`版本的推出，我们含泪挥别[`IDE Eval Reset`](https://zhile.io/2020/11/18/jetbrains-eval-reset-deprecated.html)

> 可是，知了大佬技艺日渐精进，魔高一丈，最终祭出大杀器————[ja-netfilter](https://zhile.io/2021/11/29/ja-netfilter-javaagent-lib.html) !!!

> 接下来，本文将带大家一起见证它的牛逼。。。。 

> 最后，本文只做个人学习研究之用，不得用于商业用途！
> 建议[点击链接](https://www.jetbrains.com/idea/buy/#commercial) 购买正版



## 准备

- 官方下载相关产品（推荐下载 [ToolBox](https://www.jetbrains.com/toolbox-app/) 进行相关产品安装，便于后期软件及软件管理升级）

   <img src="https://s1.ax1x.com/2022/04/27/LLSuTS.png" width="50%"> 
- 访问[https://jetbra.in/s](https://jetbra.in/s) 下载 [ja-netfilter-all.zip](https://jetbra.in/files/ja-netfilter-all-7d68b53deb1b1a16f8e95ecf8f3f98805d18368d.zip)
- 解压ja-netfilter-all.zip

## 开始

> 两种进行路径。

#### 第一种：

1. 通过注册帐号并登陆进入试用期，打开idea。或在[https://www.ajihuo.com/idea/4222.html](https://www.ajihuo.com/idea/4222.html) 该站获取激活码，打开idea。

2. 打开idea菜单`Help`->`Edite Custom VM Options...`,对idea启动脚本进行编辑，在文件下面添加一行:

   `-javaagent:/你/的/路/径/ja-netfilter-all/ja-netfilter.jar`

   如图：

   <img src="https://s1.ax1x.com/2022/04/27/LLCcmq.png" width="100%">  



3.   重启idea，在 [https://jetbra.in/s](https://jetbra.in/s) 复制对应的激活码，输入注册码，激活成功。

---

#### 第二种 `该方法不一定能成功`

1. 进入文件夹ja-netfilter-all/scripts/，执行系统对应的脚本(windows执行.vbs后缀的,mac及linux执行.sh后缀的)。
2. 启动idea,在[https://jetbra.in/s](https://jetbra.in/s)复制对应的激活码，输入注册码，激活成功。


## 其他
 * 编辑`~/ja-netfilter-all/config/mymap.conf`可随意修改到期时间及licenseeName。
 * 该软件适用所有插件


## 致谢
 * 再次致谢知了[https://zhile.io/](https://zhile.io/)


