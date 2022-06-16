---
layout: post
title: Navicat16 破解（重置试用）
date: 2022-05-12 12:10:05
tags:
- 工具
categories:
- [技术,工具]
description: 适用与linux/mac版本重置试用，windows版本可参考
---

### 1.删除Navicat注册表文件 

最简单粗暴的办法是直接备份移除dconf文件。

`简单来说，dconf 是一种基于键的配置存储系统，有点类似于 Windows 下的注册表。`
```bash
mv .config/dconf/user .config/dconf/user_b
```
直接移除固然简单，但有可能造成其他软件的一些配置的重置。
所以更文雅的做法是编辑他，指定navicat的注册表进行删除重置。
dconf文件，可使用dconf命令直接编辑，本文推荐使用各自系统对应的图形编辑器，
以作者的arch系统为例，对应图形编辑器是dconf-editor，安装使用即可，其他发行版可自行搜索。

安装与启动：
```bash
pacman -S dconf-editor&&dconf-ecditor
```


<img src="https://s1.ax1x.com/2022/05/12/O0MYuT.png" width="70%">    


启动后右键选中重置即可。

### 2.删除navicat配置文件or编辑navicat配置文件

```bash
rm -rf ~/.config/navicat/Premium/preferences.json
```
不过该方法会直接把原本本地配置的链接或密码删除，打开navicat后需要重新配置链接。
所以这里可以编辑preferences.json文件，删除json的第一个hash码字段即可。



### 最后
通过学习dconf文件的编辑。我们可以解锁更多玩法，本文仅供学习交流，有能力请尽快入正。