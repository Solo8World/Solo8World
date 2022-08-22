
---
title: Arch系统磁盘空间清理
date: 2022-08-22 13:33:59
tags:
- arch
- linux
categories:
- [技术,linux]
toc: true
description: 某日公司办公电脑磁盘空间告警，遂进行清理，记录一下
---

### pacman/yay 无用软件包清理 

`大概5GB`

清理旧版本包及已卸载的包：
 ```
pacman -Scc
 ```

清理孤立的未使用的包：
 ```
pacman -R (pacman -Qdtq)
 ```


### systemd日志文件清理与存储设置  

`大概4GB`


推荐：设置最大存储时间：

 ```
journalctl --vacuum-time=1months
 ```


设置最大存储量：

 ```
journalctl --vacuum-size=50M
 ```
（按需选择其一即可）