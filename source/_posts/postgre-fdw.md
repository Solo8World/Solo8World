---
title: PG下常用的自建函数
date: 2022-06-03 00:22:48
tags:
- postgres
categories:
- [技术]
toc: true
description: postgres fdw是一种外部访问接口，它可以被用来访问存储在外部的数据，这些数据可以是外部的pg数据库，也可以oracle、mysql、mongo、redis等数据库，甚至可以是文件。
---


## 简介

postgres fdw是一种外部访问接口，它可以被用来访问存储在外部的数据，这些数据可以是外部的pg数据库，也可以oracle、mysql、mongo、redis等数据库，甚至可以是文件。

截止到目前为止, 全世界已有了成百上千种数据源有了相应的FDW实现, 从传统的文件系统到各种新型的Nosql数据库, 甚至还包括互联网上的Web Service。

目前支持的fdw外部数据源：https://wiki.postgresql.org/wiki/Foreign_data_wrappers

使用FDW的核心就在于使用外部表。尽管面向不同数据源的FDW实现各有不同, 但是受益于SQL/MED定义的标准, 创建不同数据源的外部表的方法都是一样的, 这里我们拿mysql_fdw做示例。


## 实现原理

`执行sql访问外部数据源`->`sql解析`->`下推至外部数据源执行`->`返回数据结果`


## 安装

```
--向PG安装某个数据源的FDW扩展
CREATE EXTENSION mysql_fdw;
--drop EXTENSION mysql_fdw;

--创建该数据源的FDW对象
CREATE FOREIGN DATA WRAPPER mysql_fdw;
--DROP  FOREIGN DATA WRAPPER mysql_fdw;
```

## 使用

#### 1.创建该数据源的服务器对象

```
--创建服务连接信息
CREATE SERVER erp_server
     FOREIGN DATA WRAPPER mysql_fdw
     OPTIONS (host '172.16.0.70', port '3306');
--DROP SERVER erp_server;

--建立用户映射 `bitest`当前pg用户，`rwbj`外部数据源用户
CREATE USER MAPPING FOR bitest
SERVER erp_server
OPTIONS (username 'rwbj', password '22eaa714ae0b49e09808233f3d3d6212');

--DROP USER MAPPING FOR bitest SERVER erp_server
```

#### 2. 创建外部表（三种方式）
```
--创建外部数据表（指定字段）
CREATE FOREIGN TABLE erp_contract (id bigint) server erp_server
    options (dbname 'erp', table_name 'erp_contract');

--除了指定字段创建外部表以外，还可以直接导入指定表的所有字段
import foreign schema erp limit to (erp_rent_shop,erp_sys_user) from server erp_server into bi;


-- 以及，一次性导入指定DB下的所有表
import foreign schema erp from server erp_server into bi;
```

#### 3. 使用外部表
```
--然后，像对自家表一样操作外部数据源mysql
select * from erp_contract;
insert into erp_contract (id) values (1);
update erp_contract set id= 1 where id = 1;
delete from erp_contract where id  = 1;
```

#### 4. 删除外部表
```
-- 指定表名称，删除多个外部表
drop foreign table bi.erp_contract, bi.erp_rent_shop;

-- 或者构造sql语句批量删除
select
	'drop foreign table ' || t.table_schema || '.' || t.table_name || ';' as drop_sql
from information_schema.tables t
where t.table_type in ('FOREIGN')
	and t.table_schema in ('bi');

```

#### 5.扩展

由FDW实现原理可知，PG本身没有对外部数据源进行存储管理（也不可能），每次访问都是通过下推到外部数据库进行的执行。对于读数据频次较大、或数据量较多、或复杂查询的场景（比如BI统计），我们可以通过物化外部数据来减轻对外部数据源的压力和执行开销，如下：

```
--创建物化视图
CREATE MATERIALIZED VIEW erp_contract_view as select * from erp_contract;
--定期刷新物化视图
REFRESH MATERIALIZED VIEW erp_contract_view;
```

也可以创建pg本地表来对第三方数据源的数据进行定期增量同步：
```
--创建pg本地表
CREATE TABLE  erp_contract_t as select * from erp_contract;
--同步数据
DELETE
FROM erp_contract_t t
WHERE EXISTS(SELECT 1 FROM erp_contract f WHERE t.id = f.id);
INSERT INTO erp_contract_t      SELECT * FROM erp_contract;
```


## 总结

之前一直没有使用过pg的fdw扩展,因为EAM的数据可视化需求的实现方案涉及到了，所以去了解试用了一下。

方便规范，整体可用，直接免去了在星汉内部系统中数据共享与交换的开发成本。

但该方案跟育铜讨论后，育铜提出了后期维护与管理不透明的主张。确是如此，我明白后期维护与管理的弊端。

但我没想明白的是：如果如此的话，FDW的存在还有什么意义呢？也许FDW还需要一个web管理？我不知道Pigsty是否对此作了支持，后面去看一下。

