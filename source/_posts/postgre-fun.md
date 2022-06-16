---
title: PG下常用的自建函数
date: 2022-06-03 00:22:48
tags:
- postgres
categories:
- [技术]
toc: true
description: 适度的抽取逻辑来组装函数，有助于提升开发效率及后期维护的便捷性,本文列举了本人在BI开发过程中所封装的函数供大家参考。
---

> 适度的抽取逻辑来组装函数，有助于提升开发效率及后期维护的便捷性,本文列举了本人在BI开发过程中所封装的函数供大家参考。

### 数值计算类

* 除法计算，保留两位小数

```sql
create function divide(divisor numeric, divisor2 numeric) returns numeric
    language plpgsql
as
$$
BEGIN
  -- 若除数等于0或为null 直接返回0.00
if  divisor2 = 0  or  divisor2 is null then
    RETURN 0.00;
end if;

--round结果保留两位
  RETURN COALESCE(round(COALESCE(divisor/divisor2,0),2),0);
END
$$;

```



* 除法计算，自定义精度

```sql
create function divide_scale(divisor numeric, divisor2 numeric, round_scale integer) returns numeric
    language plpgsql
as
$$
BEGIN
  -- 除法函数自定义精度...
  -- 若除数等于0或为null 直接返回0
if  divisor2 = 0  or  divisor2 is null then
    RETURN round(0,round_scale);
end if;

  RETURN round(COALESCE(divisor/NULLIF(divisor2,0),0),round_scale);
END
$$;
```




* 百分比比值计算,保留两位小数

```sql
create function percent_prop(divisor numeric, divisor2 numeric) returns numeric
    language plpgsql
as
$$
BEGIN
  -- 若除数等于0或为null 直接返回0
if  divisor2 = 0  or  divisor2 is null then
    RETURN 0.00;
end if;

  RETURN round(round(COALESCE(divisor/NULLIF(divisor2,0),0),4)*100,2);
END
$$;

```


### 数组处理类


* 数组去重

```sql
create function array_distinct(anyarray) returns anyarray
    language sql
as
$$
SELECT ARRAY(SELECT DISTINCT unnest($1));
$$;
```


* 数组最大值

```sql
create function array_max(anyarray) returns anyelement
    language sql
as
$$
SELECT max(x) FROM unnest($1) as x;
$$;
```




* 数组最小值

```sql
create function array_min(anyarray) returns anyelement
    language sql
as
$$
SELECT min(x) FROM unnest($1) as x;
$$;
```



### 更新被依赖对象DDL工具函数

>我们在修改视图定义时，如果该视图有被其他视图或函数依赖的话，是无法直接删除并更新的，只能找到所有依赖该视图的对象，依次备份后删除才能对该视图进行更新。随着业务的复杂，依赖关系也变的复杂，这项工作比较耗时耗力，所以就有了该函数，自动查询所有依赖，对依赖DDl保存并自动删除。更新该视图后，再次执行函数，自动对依赖进行恢复。


```sql
--依赖DDL暂存表
create table deps_saved_ddl
(
    deps_id          integer default nextval('deps_saved_ddl_deps_id_seq'::regclass) not null
        primary key,
    deps_view_schema name,
    deps_view_name   name,
    deps_ddl_to_run  text
);

--自动查询依赖并保存后删除函数
create function deps_save_and_drop_dependencies(p_view_schema name, p_view_name name) returns void
    language plpgsql
as
$$
declare
  v_curr record;
begin
for v_curr in 
(
  select obj_schema, obj_name, obj_type from
  (
  with recursive recursive_deps(obj_schema, obj_name, obj_type, depth) as 
  (
    select p_view_schema, p_view_name, null::char, 0
    union
    select dep_schema::name, dep_name::name, dep_type::char, recursive_deps.depth + 1 from 
    (
      select ref_nsp.nspname ref_schema, ref_cl.relname ref_name, 
	  rwr_cl.relkind dep_type,
      rwr_nsp.nspname dep_schema,
      rwr_cl.relname dep_name
      from pg_depend dep
      join pg_class ref_cl on dep.refobjid = ref_cl.oid
      join pg_namespace ref_nsp on ref_cl.relnamespace = ref_nsp.oid
      join pg_rewrite rwr on dep.objid = rwr.oid
      join pg_class rwr_cl on rwr.ev_class = rwr_cl.oid
      join pg_namespace rwr_nsp on rwr_cl.relnamespace = rwr_nsp.oid
      where dep.deptype = 'n'
      and dep.classid = 'pg_rewrite'::regclass
    ) deps
    join recursive_deps on deps.ref_schema = recursive_deps.obj_schema and deps.ref_name = recursive_deps.obj_name
    where (deps.ref_schema != deps.dep_schema or deps.ref_name != deps.dep_name)
  )
  select obj_schema, obj_name, obj_type, depth
  from recursive_deps 
  where depth > 0
  ) t
  group by obj_schema, obj_name, obj_type
  order by max(depth) desc
) loop

  insert into deps_saved_ddl(deps_view_schema, deps_view_name, deps_ddl_to_run)
  select distinct p_view_schema, p_view_name, indexdef
  from pg_indexes
  where schemaname = v_curr.obj_schema
  and tablename = v_curr.obj_name;

  insert into deps_saved_ddl(deps_view_schema, deps_view_name, deps_ddl_to_run)
  select distinct tablename, rulename, definition
  from pg_rules
  where schemaname = v_curr.obj_schema
  and tablename = v_curr.obj_name;

  insert into deps_saved_ddl(deps_view_schema, deps_view_name, deps_ddl_to_run)
  select p_view_schema, p_view_name, 'COMMENT ON ' ||
  case
  when c.relkind = 'v' then 'VIEW'
  when c.relkind = 'm' then 'MATERIALIZED VIEW'
  else ''
  end
  || ' ' || n.nspname || '.' || c.relname || ' IS ''' || replace(d.description, '''', '''''') || ''';'
  from pg_class c
  join pg_namespace n on n.oid = c.relnamespace
  join pg_description d on d.objoid = c.oid and d.objsubid = 0
  where n.nspname = v_curr.obj_schema and c.relname = v_curr.obj_name and d.description is not null;

  insert into deps_saved_ddl(deps_view_schema, deps_view_name, deps_ddl_to_run)
  select p_view_schema, p_view_name, 'COMMENT ON COLUMN ' || n.nspname || '.' || c.relname || '.' || a.attname || ' IS ''' || replace(d.description, '''', '''''') || ''';'
  from pg_class c
  join pg_attribute a on c.oid = a.attrelid
  join pg_namespace n on n.oid = c.relnamespace
  join pg_description d on d.objoid = c.oid and d.objsubid = a.attnum
  where n.nspname = v_curr.obj_schema and c.relname = v_curr.obj_name and d.description is not null;
  
  insert into deps_saved_ddl(deps_view_schema, deps_view_name, deps_ddl_to_run)
  select p_view_schema, p_view_name, 'GRANT ' || privilege_type || ' ON ' || table_schema || '.' || quote_ident(table_name) || ' TO ' || grantee
  from information_schema.role_table_grants
  where table_schema = v_curr.obj_schema and table_name = v_curr.obj_name;
  
  if v_curr.obj_type = 'v' then
    insert into deps_saved_ddl(deps_view_schema, deps_view_name, deps_ddl_to_run)
    select p_view_schema, p_view_name, 'CREATE VIEW ' || v_curr.obj_schema || '.' || quote_ident(v_curr.obj_name) || ' AS ' || view_definition
    from information_schema.views
    where table_schema = v_curr.obj_schema and table_name = v_curr.obj_name;
  elsif v_curr.obj_type = 'm' then
    insert into deps_saved_ddl(deps_view_schema, deps_view_name, deps_ddl_to_run)
    select p_view_schema, p_view_name, 'CREATE MATERIALIZED VIEW ' || v_curr.obj_schema || '.' || quote_ident(v_curr.obj_name) || ' AS ' || definition
    from pg_matviews
    where schemaname = v_curr.obj_schema and matviewname = v_curr.obj_name;
  end if;
  
  execute 'DROP ' ||
  case 
    when v_curr.obj_type = 'v' then 'VIEW'
    when v_curr.obj_type = 'm' then 'MATERIALIZED VIEW'
  end
  || ' ' || v_curr.obj_schema || '.' || quote_ident(v_curr.obj_name);
  
end loop;
end;
$$;


-- 依赖恢复函数
create function deps_restore_dependencies(p_view_schema name, p_view_name name) returns void
    language plpgsql
as
$$
declare
  v_curr record;
begin
for v_curr in 
(
  select deps_ddl_to_run 
  from deps_saved_ddl
  where deps_view_schema = p_view_schema and deps_view_name = p_view_name
  order by deps_id desc
) loop
  execute v_curr.deps_ddl_to_run;
end loop;
delete from deps_saved_ddl
where deps_view_schema = p_view_schema and deps_view_name = p_view_name;
end;
$$;

```






