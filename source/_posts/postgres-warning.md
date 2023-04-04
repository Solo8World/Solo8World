---
title: PG实践踩坑记录
date: 2023-01-31 17:57:30
tags:
- postgres
categories:
- [技术]
description: 记录一下找bug过程中与思维惯性相悖的千奇百怪,不间断更新
toc: true
---

> 记录一下找bug过程中与思维惯性相悖的千奇百怪,不间断更新

---
### ID自增序列
  * 你有一张id自增的表，id自增至99。

  * A同事执行了指定id值的插入语句 `INSERT INTO XXX （id，value） VALUES（100，'李大本事'）；`

  * B同事再执行不指定id值的插入语句 `INSERT INTO XXX （value） VALUES（'李白'）；`

  * 此时报错`id already exists`
            
  请执行以下sql后再次执行插入：
   
```sql
     SELECT setval('bi_data_sale_order_detail_id_seq', 
                   (SELECT MAX(id) FROM bi_data_sale_order_detail));
```

      
----



### 数值的类型与保留位数

  根据产品需求，你需要一个3与13的百分比比值，要求结果（比值）保留两位小数。

  `select round(3/13,4)*100；`执行这个sql你只会得到0；
 
  `select round(3::numeric/13::numeric,4)*100；`执行这个sql，你大概率会得到一个肉眼看起来是23.08的数字,直到你发现程序bug前这不会有什么问题。

  `select ((round(3::numeric/13::numeric,4))*100)::varchar；`再次执行这个sql，你才能看到这个结果的真正面目---他其实是‘23.0800’；

  最终，你只能这么干，  `select round(round(3::numeric/13::numeric,4)*100,2);`

----


### DataGrip与array_agg与时区

这个坑只在DataGrip有复现，指定数据库连接参数-Duser.timezone=PRC，执行以下sql。

```sql
     with test as (select  now() as d)
     select array_agg(d),d from test group by d;
```

  会发现两个d值发生了时区不一致。
  
  只能转为varchar来规避该问题。

```sql
     with test as (select  now() as d)
     select array_agg(d::varchar),d from test group by d;
```

----


### week的 date_part 与date_trunc

根据产品需求按周统计数据：

```sql
select date_part('year'::text, order_date) AS year,
       date_part('week'::text, order_date) AS week
from xxx
group by date_part('year'::text, order_date) AS year, date_part('week'::text, order_date) AS week
```
直到执行以下sql：

```sql
select   date_part('year'::text,'2022-01-01'::date)                                       AS year,
            date_part('week'::text,'2022-01-01'::date)                                       AS week
```

应修改为：

```sql
select   date_part('year'::text, date_trunc('week'::text,'2022-01-01'::date) )    AS year,
            date_part('week'::text,'2022-01-01'::date)                                           AS week
```


----


### week的计算 与ISO8601标准

一直粗暴理解pg的周计算标准是以第一个周一来算第一周：

直到执行以下sql：

```sql
SELECT date_part('WEEK','2020-01-01'::date);
SELECT date_part('WEEK','2022-01-01'::date);
```
查阅相关资料后发现，周数计算是按照ISO8601标准来的：

ISO 8601中关于星期规定如下：
该年度的1月1日，如果是星期一，星期二，星期三，星期四，那么，1月1日所在的星期就是该年度的第一个星期。
该年度的1月1日，如果是星期五，星期六，星期日，那么1月1日所在的星期就是上一年的最后一个星期。
每个星期以星期一为开始，星期日为结束。


  









