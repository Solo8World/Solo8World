---
title: Java三页代码之——实现Mysql透明加解密
date: 2020-08-16 15:14:18
tags: 
- java
- x页代码
- 代码
categories:
- 技术
- 代码
toc: true
description: 前段时间有项目需求做个简单的数据加密，根据开发成本和程序性能综合考虑，最终选定利用mysql加解密函数进行实现。
---
## 前言
前段时间有项目需求做个简单的数据加密，根据开发成本和程序性能综合考虑，
最终选定利用mysql加解密函数进行实现。

## 实现思路
项目sql执行前进行拦截，如果包含加密字段则对sql进行字符串截取拼接处理，
不同类型不同处理方式：  
* select类型：select字段拼上解密函数及密钥
* insert类型：insert对应字段值拼上加密函数及密钥
* delete类型：对应字段where条件值拼上加密函数及密钥
* update类型：update对应字段值拼上加密字段及密钥，对应字段where条件值拼上加密函数及密钥

## 用到的第三方工具包
[HuTool](https://hutool.cn/) (一个小而美的java工具包)  
```xml
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.4.3</version>
</dependency>
```
## 上代码
* 配置类
```java
package com.lee.blog.config.mybatis;

import java.util.Arrays;
import java.util.List;

/**
 * @author lizhuo
 */
public class EncConfig {
    //要加密的字段
    public static final List<String> FIELD = 
                  Arrays.asList("card_no", "card_key", "card_private_no");

    public static final String SALT = "\"bc7a5744-fbf0-5288-c20e-e87ea9ee1ba0\"";

    public static final String ENC_PREFIX = "HEX(ENCODE(";
    public static final String ENC_SUFFIX = "," + SALT + "))";

    public static final String DEC_PREFIX = "DECODE(UNHEX(";
    public static final String DEC_SUFFIX = ")," + SALT + ")";

    public static final String AS = "AS";
    public static final String SELECT = "SELECT";
    public static final String FROM = "FROM";

}
```
* mybatise拦截类
```java 
package com.lee.blog.config.mybatis;


import cn.hutool.core.util.ReflectUtil;
import cn.hutool.core.util.StrUtil;
import org.apache.commons.lang3.StringUtils;
import org.apache.ibatis.executor.statement.RoutingStatementHandler;
import org.apache.ibatis.executor.statement.StatementHandler;
import org.apache.ibatis.mapping.BoundSql;
import org.apache.ibatis.plugin.*;
import org.jetbrains.annotations.NotNull;

import java.sql.Connection;
import java.util.*;
import java.util.concurrent.atomic.AtomicReference;


/**
 * @author lizhuo
 */
@Intercepts({
        @Signature(type = StatementHandler.class, method = "prepare", args = {Connection.class, Integer.class})
})
public class PrepareInterceptor implements Interceptor {

    private static final String INSERT = "INSERT";

    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }

    @Override
    public void setProperties(Properties properties) {
    }

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        if (invocation.getTarget() instanceof RoutingStatementHandler handler) {
            StatementHandler delegate =
                    (StatementHandler) ReflectUtil.getFieldValue(handler, "delegate");
            BoundSql boundSql = delegate.getBoundSql();
            final String sql = boundSql.getSql();
            if (needHandler(sql)) {
                handlerSql(boundSql);
            }
        }
        return invocation.proceed();
    }

    private void handlerSql(BoundSql boundSql) {
        //String result = SQLUtils.format(boundSql.getSql(), JdbcConstants.MYSQL);
        String sql = replaceSql(boundSql.getSql());
        switch (sql.substring(0, 6).toUpperCase()) {
            case EncConfig.SELECT -> sql = handlerQuerySql(sql);
            case INSERT -> sql = handlerInsertSql(sql);
            default -> sql = handlerDeleteOrUpdateSql(sql);
        }
        ReflectUtil.setFieldValue(boundSql, "sql", sql);
    }

    private String handlerDeleteOrUpdateSql(String sql) {
        return new Sql(sql).toString();
    }

    private String handlerQuerySql(String sql) {
        return new Sql(sql).toString();
    }


    private String handlerInsertSql(String sql) {
        final List<String> insertFiled = Arrays.asList(
                sql.substring(sql.indexOf("(") + 1, sql.indexOf(")")).split(StrUtil.COMMA));
        String insertValueStr = sql.substring(sql.lastIndexOf("(") + 1, sql.lastIndexOf(")"));
        List<String> insertValue = new ArrayList<>(Arrays.asList(
                insertValueStr.split(StrUtil.COMMA)));

        for (int i = 0; i < insertFiled.size(); i++) {
            final Optional<String> any = EncConfig.FIELD.stream().filter(insertFiled.get(i)::contains).findAny();
            if (any.isPresent()) {
                insertValue.set(i, EncConfig.ENC_PREFIX + insertValue.get(i) + EncConfig.ENC_SUFFIX);
            }
        }
        return replaceSql(sql, insertValueStr, insertValue);
    }

    @NotNull
    private String replaceSql(String sql, String resourceStr, List<String> targetStrList) {
        AtomicReference<String> targetQueryFiledStr = new AtomicReference<>("");
        targetStrList.forEach(i -> targetQueryFiledStr.set(
                StringUtils.isBlank(targetQueryFiledStr.get()) ?
                        targetQueryFiledStr + i : targetQueryFiledStr + StrUtil.COMMA + i));
        sql = sql.replace(resourceStr, targetQueryFiledStr.get());
        return sql;
    }

    private boolean needHandler(String sql) {
        return EncConfig.FIELD.stream().anyMatch(sql::contains);
    }

    /**
     * 过滤特殊字符
     */
    private String replaceSql(String sql) {
        sql = sql.replaceAll("\\n", " ");
        sql = sql.replaceAll("\\t", " ");
        return sql;
    }

}
```

* sql转换类
```java
package com.lee.blog.config.mybatis;

import cn.hutool.core.util.StrUtil;
import lombok.Data;
import org.apache.commons.collections4.CollectionUtils;
import org.apache.commons.lang3.StringUtils;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

import static  com.lee.blog.config.mybatis.EncConfig.*;

/**
 * @author lizhuo
 */
@Data
public class Sql {


    private String execType;

    private List<QueryField> queryField;

    private String remainder;


    public Sql(String sqlStr) {
        StringBuilder sql = new StringBuilder(sqlStr);
        this.execType = sql.substring(0, 6);

        if (!SELECT.equalsIgnoreCase(execType)) {
            this.remainder = sql.substring(6);
            return;
        }
        int fromIndex = StringUtils.indexOfIgnoreCase(sql, FROM);
        //解析并赋值对象
        String queryFieldStr = sql.substring(6, fromIndex);
        boolean has = EncConfig.FIELD.stream().anyMatch(queryFieldStr::contains);
        if (has) {
            List<String> queryField = Arrays.asList(queryFieldStr.split(","));
            List<QueryField> field = new ArrayList<>(queryField.size());
            queryField.forEach(i -> field.add(new QueryField(i)));
            this.queryField = field;
        } else {
            this.queryField = Collections.singletonList(new QueryField(queryFieldStr));
        }
        this.remainder = sql.substring(fromIndex);
    }

    @Override
    public String toString() {
        StringBuilder sql = new StringBuilder(execType);

        if (CollectionUtils.isNotEmpty(queryField)) {
            queryField.forEach(i -> sql.append(i.toString()).append(","));
            sql.setCharAt(sql.length() - 1, " ".charAt(0));
        }

        EncConfig.FIELD.forEach(i -> {
            if (remainder.contains(i + " = ?")) {
                remainder = remainder.replace(i + " = ?",
                        i + " = " + EncConfig.ENC_PREFIX + "?" + EncConfig.ENC_SUFFIX);
            }
            if (remainder.contains(i + " in") || remainder.contains(i + " not in")) {

                String substring = remainder.substring(
                        remainder.indexOf(i + " in (") + (i + " in (").length());

                String valueStr = substring.substring(0, substring.indexOf(")"));

                remainder = remainder.replace(valueStr,
                        valueStr.replace("?",
                                EncConfig.ENC_PREFIX + "?" + EncConfig.ENC_SUFFIX));
            }
        });
        sql.append(remainder);
        return sql.toString();
    }


    @Data
    static class QueryField {

        private String field;

        private String function;

        private String alias;

        public QueryField(String field) {
            boolean anyMatch = EncConfig.FIELD.stream().anyMatch(field::contains);
            if (!anyMatch) {
                this.field = field;
                return;
            }
            boolean hasFunction = field.contains("(");
            if (hasFunction) {
                int lastIndexOfStart = field.lastIndexOf("(");
                int lastIndexOfEnd = field.lastIndexOf(")");
                this.function = field.substring(0, lastIndexOfStart) + "(@function@)"
                        + field.substring(lastIndexOfEnd + 1);

                field = field.substring(lastIndexOfStart + 1, lastIndexOfEnd);
            }
            boolean hasAlias = StringUtils.containsIgnoreCase(field, AS);
            boolean hasPrefix = field.contains(StrUtil.DOT);
            if (hasAlias) {
                this.alias = field.substring(StringUtils.indexOfIgnoreCase(field, AS) + 2);
                field = field.substring(0, StringUtils.indexOfIgnoreCase(field, AS));
            } else {
                this.alias = AS + " " + (hasPrefix ?
                        field.substring(field.indexOf(StrUtil.DOT) + 1) : field);
            }
            this.field = field;
        }

        @Override
        public String toString() {
            boolean anyMatch = EncConfig.FIELD.stream().anyMatch(field::contains);
            field = anyMatch ? EncConfig.DEC_PREFIX + field + EncConfig.DEC_SUFFIX : field;

            return StringUtils.isEmpty(function) ? field + (alias == null ? "" : alias)
                    : function.replace("@function@", field) + (alias == null ? "" : alias);
        }
    }
}
```


_ps_ ：
* 拦截处理并不支持SELECT * 语句,不建议项目中使用SELECT *； 

* 项目中如果没有单独mybatis配置类的话，拦截器默认生效，如果包含配置类则需在配置类中指定开启

