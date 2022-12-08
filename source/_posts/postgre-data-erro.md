---
title: 主机暴毙，PG驾崩，视图数据损坏修复过程
date: 2022-12-07 20:30:19
tags:
- postgres
categories:
- [技术]
description: 测试服务器又又又突然挂了，重启机器后pg又又未能重启，一番操作后，成功启动。但成功启动后发现部分物化视图查询报错...
---

```
 ERROR:  could not open file "base/67201/11128463": No such file or directory
```

```
 ERROR:  cache lookup failed for relation 11128951
```

## 背景
测试环境postgreSQL数据库是以docker容器部署的单节点服务，机器在本地机房里。

测试服务器又又又突然挂了，重启机器后pg又又未能重启，参照[记录机器断电后PG故障的解决](https://leeblog.icu/2022/05/24/)
解决后，成功启动。

但成功启动后发现部分物化视图查询报错
```
could not open file "base/67201/11128463": No such file or directory
```

## 问题解决
* 猜测是由于服务器挂掉的时候，物化视图正在执行更新操作，突然断开导致视图数据文件损坏。

* 遂尝试刷新物化视图，以期待能够重建数据文件
    ```sql
    refresh materialized view  bi_view_real_time_sale_period_market_all;
    ```
  不曾想又报了一错：
    ```
    [XX000] ERROR: cache lookup failed for relation 11128466
    ```

* 可想而知该视图已被锁定
  尝试删除视图，失败，同样的锁定错误。
* 遂进行引用删除（注意备份视图创建语句）

  * 查询视图oid
    ```sql
    select oid, relname,relfilenode from pg_class where relname like '%bi_view_real_time_sale_period_market_all%';
    
    row: 1066610,bi_view_real_time_sale_period_market_all
    ```

  * 删除引用
    ```sql
    delete from pg_class where oid=1066610;
    
    delete from pg_depend where objid=1066610;
    
    delete from pg_depend where refclassid=1066610;
    
    delete from pg_depend where refobjid=1066610;
    ```
  * 删除type
    ```sql
    drop type  bi_view_real_time_sale_period_market_all;
    ```
  * 重建视图,执行成功

