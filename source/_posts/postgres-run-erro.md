---
title: 记录一次机房停电后PG故障的解决
date: 2022-05-24 19:25:48
tags:
- postgres
categories:
- [技术]
description: 测试环境postgreSQL数据库是以docker容器部署的单节点服务，机器在本地机房里。
---

```azure
2022-05-24 11:39:31.957 CST [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
2022-05-24 11:39:31.957 CST [1] LOG:  listening on IPv6 address "::", port 5432
2022-05-24 11:39:31.957 CST [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2022-05-24 11:39:31.960 CST [24] LOG:  database system was interrupted; last known up at 2022-05-24 07:18:02 CST
2022-05-24 11:39:31.961 CST [24] LOG:  invalid primary checkpoint record
2022-05-24 11:39:31.961 CST [24] PANIC:  could not locate a valid checkpoint record
2022-05-24 11:39:43.952 CST [25] FATAL:  the database system is starting up
2022-05-24 11:39:44.719 CST [26] FATAL:  the database system is starting up
2022-05-24 11:39:45.663 CST [27] FATAL:  the database system is starting up
2022-05-24 11:39:48.726 CST [1] LOG:  startup process (PID 24) was terminated by signal 6: Aborted
2022-05-24 11:39:48.726 CST [1] LOG:  aborting startup due to startup process failure
2022-05-24 11:39:48.898 CST [1] LOG:  database system is shut down
```
## 前情提要
测试环境postgreSQL数据库是以docker容器部署的单节点服务，机器在本地机房里。

昨日凌晨机房又一次突然断电，断电本是机房常事，奈何这次机房服务器再次恢复后pg的容器未能成功重启。

运维无策，大群求助。我一看这还了得，毕竟辛苦一周的工作成果还在数据库里尚未备份，可不敢怠慢。

果断出手，连接vpn，登陆服务器。

## 问题解决  
* `docker ps` 可以看到，pg的容器还在挣扎着不断启动、失败、启动、失败。
`docker logs -f --tail 200 pg` 查看容器启动日志，发现情况还不算严重。应是断电时pg正在进行事务操作，突然断电导致事务日志文件损坏，pg再次启动时读取失败。此时应该只需进入pg容器里重置事务日志即可。
然而是容器部署的，容器始终无法启动成功，根本没有下手执行重置的时机。

 <img src="https://s1.ax1x.com/2022/05/26/XEIbo6.png" width="100%">  

* 这机器的服务部署并非出自我手，为了搞清敌情，`docker inspect pg`，查看容器详情。
通过inspect可以看到，容器的数据存储位置映射了外部的文件位置，而且在环境变量中指定了数据文件存储位置。
<img src="https://s1.ax1x.com/2022/05/26/XVdvX4.png" width="100%">

* 当鸡立断，决定启动一个临时容器映射同样的文件存储位置来执行日志重置。
   * 先是一个`docker update --restart=no pg`，叫停当前容器的不断重启行为，防止启动临时容器后两个容器同时读写日志文件造成场面进一步的混乱。
   *  然后基于原来容器的镜像，以临时模式启动一个新容器，并进入容器bash端，切换postgres用户（默认必须以postgres执行），找到rest工具位置全路径，
       ctrl c ，ctrl v，回车，
     <img src="https://s1.ax1x.com/2022/05/26/XVwPtx.png" width="100%">
     <img src="https://s1.ax1x.com/2022/05/26/XVwA1O.png" width="100%">

* 执行成功。退出当前容器，因为是临时的所以退出即运行终止。
  
* `docker start pg`  再次启动原来的容器，启动成功。

有惊无险，打完收工，没有备份，下次一定。




## 总结:
使用到的命令按执行顺序如下:
```bash
docker ps -a                          #查看容器列表
docker logs -f --tail 200 pg          #查看pg容器启动日志
docker inspect pg                     #查看容器配置详情
docker update --restart=no pg         #取消失败自动重启
docker run -it  -v   /opt/appdata/pgdata:/var/lib/postgresql/data  --env PGDATA=/var/lib/postgresql/data/pgdata  postgres:13   /bin/bash  ##启动临时容器并进入bash
su postgres                          #临时容器内:切换postgres用户
pg_resetwal -f /var/lib/postgresql/data/pgdata  #临时容器内: 利用pg工具，执行事务日志重置
exit                                 #临时容器内:退出容器
docker start pg                      #启动pg容器
```