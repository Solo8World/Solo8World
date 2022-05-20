---
title: Java一页代码之——实现微信自动智能回复
date: 2020-07-10 15:25:48
tags:
- java
- x页代码
- wechat
- 代码
categories:
- 技术
- 代码
toc: true 
description: 机器可以替代我们自动打开空调，可以替代我们洗碗拖地，甚至替代我们开车(真开车，别多想)，现在，它还可以替我们聊天，是的，它说：Yes，I do!
---

## 前言

> 机器可以替代我们自动打开空调，可以替代我们洗碗拖地，甚至替代我们开车(真开车，别多想)，
现在，它还可以替我们聊天，是的，它说：Yes，I do!

> 可喜可贺的是，帮我回复女朋友微信消息的智能程序写好了，女朋友没了。

> 所以屏幕前好奇这个程序的你，还看什么代码，赶紧滚去找对象。

> 有对象了？那我劝你且行且珍惜。本程序对运行后果概不负责。

## 正题
### 实现效果先看一眼：
【这个enmm暂时还没有，因为木有人给俺发消息，如果再给俺一次重来的机会，害...】 

### 实现思路：
1. 利用微信网页端接口登陆微信并监听消息
2. 收到消息后请求图灵智能机器人api接口获得智能回复消息内容
3. 通过微信接口回复智能消息

### 实现

* maven引入包
  * wechat-api (来自[biezhi的github](https://github.com/biezhi/wechat-api))
    ```xml
      <dependency>
          <groupId>io.github.biezhi</groupId>
          <artifactId>wechat-api</artifactId>
          <version>1.0.6</version>
      </dependency>
    ```
  * [HuTool](https://hutool.cn/) (一个小而美的java工具包)
    ```xml
    <dependency>
        <groupId>cn.hutool</groupId>
        <artifactId>hutool-all</artifactId>
        <version>5.4.3</version>
    </dependency>
    ```

* 一页代码（俺贴心的给你们设置了白名单机制，在哪儿设置自己找）
    ```java
    package com.lee.api;

    import cn.hutool.http.HttpRequest;

    import cn.hutool.json.JSONUtil;
    import io.github.biezhi.wechat.WeChatBot;
    import io.github.biezhi.wechat.api.annotation.Bind;
    import io.github.biezhi.wechat.api.constant.Config;
    import io.github.biezhi.wechat.api.enums.MsgType;
    import io.github.biezhi.wechat.api.model.WeChatMessage;
    import org.apache.commons.lang3.StringUtils;

    
    import java.util.Arrays;
    import java.util.List;

    
    public class HelloBot extends WeChatBot {
    
        public HelloBot(Config config) {
            super(config);
        }
    
        List<String> notRec=Arrays.asList("老聂","老黄");
        @Bind(msgType = MsgType.TEXT)
        public void handleText(WeChatMessage message) {
            if (StringUtils.isNotEmpty(message.getName())
                    &&!message.isGroup()) {
                System.out.println("接收到 ["+message.getName()+"] 的消息: "
                                                      +message.getText());
                 String mSg ="我可能有点问题,等会回复你噢";
                 if(!notRec.contains(message.getFromRemarkName())) {
                     try {
                         mSg = getReply(message.getText());
                     } catch (Exception e) {
                         System.out.println(e.getMessage());
                     }
                     this.sendMsg(message.getFromUserName(), mSg );
                 }
            }
        }
        String getReply(String msg){
    
            String param="{\n" +
                    "    \"perception\": {\n" +
                    "        \"inputText\": {\n" +
                    "            \"text\": \"MSG\"\n" +
                    "        }\n" +
                    "    },\n" +
                    "    \"userInfo\": {\n" +
                    "        \"apiKey\": \"608df9e00ab648ac91968a20b11dba3f\",\n" +
                    "        \"userId\": \"635279\"\n" +
                    "    }\n" +
                    "}";
    
            final String body = HttpRequest.post("https://openapi.tuling123.com/openapi/api/v2")
                    .body(JSONUtil.parse(param.replace("MSG",msg)).toString())
                    .execute().body();
    
            return JSONUtil.parseObj(body).getByPath("results[0].values.text").toString();
        }
    
        public static void main(String[] args) {
            new HelloBot(Config.me().autoLogin(true).showTerminal(true)).start();
        }
    }
    ```
  PS：里面图灵接口的请求参数apiKey及userId是俺在[图灵API](http://www.turingapi.com/) 注册获取的，有需要可自己去注册。
## 更多翻车栗子
[微信自动回复 | 如何智能秒回女朋友 - 程序员大本营](https://www.pianshen.com/article/96801052825/)