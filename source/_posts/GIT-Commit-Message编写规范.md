---
title: idea插件git commit template的使用与changeLog的生成
date: 2020-09-13 19:00:34
tags: 
- GIT
- 开发规范
- 工具
categories:
- [技术,工具]
toc: true
description: 一个项目一个团队的git log一定要整整齐齐的。统一的提交注释规范，除了自己赏心悦目获得极大的舒适感以外，还能为后续的code review、版本升级change log文档的生成，提供极大的便利。  

---


## 前言
git commit template 该插件是为了规范git提交注释的。一个项目一个团队的git log一定要整整齐齐的。
统一的提交注释规范，除了自己赏心悦目获得极大的舒适感以外，还能为后续的code review、版本升级change log文档的生成，提供极大的便利。  

* **[Angular规范](https://github.com/angular/angular/blob/master/CONTRIBUTING.md#-commit-message-guidelines) 提交格式如下：**

```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

* **含义：**
```
提交类型(改动涉及范围): 简要描述
<空行>
详细描述(可换多行进行描述)
<空行>
<页脚>（BREAKING CHANGE重大改动描述；Close Issue）
```

* **提交类型：**

  * feat：新功能开发
  * fix：bug修复
  * docs：项目文档改动
  * style： 代码格式（不影响代码运行的变动,空格,格式化,等等）
  * refactor：重构（理论上不影响功能的代码重构）
  * perf: 性能优化
  * test：增加或者修改测试
  * build: 影响构建系统或外部依赖项的更改(maven)
  * ci: 对CI配置文件和脚本的更改
  * chore：对非 src 和 test 目录的修改
  * revert: 还原上一次提交


* **以feat提交为例，commit message格式如下:**
```
feat(系统管理): 新增用户密码设置规则检验  

(1)验证密码设置由大小写字母数字特殊符号任意三种组合
(2)验证密码设置长度8-12位
```

* 注：
  * 提交格式中`type` 、`subject`为必填，`scope`、`body` 、`footer`为选填 
  * revert提交时以revert:开头, 后面跟着被撤销Commit的Header。  
    Body部分的格式是固定的，必须写成This reverts commit <hash>,  
    其中的hash是被撤销 commit 的 SHA 标识符.



## `Git Commit Template`插件的安装与使用
* **安装**  

idea中直接`File`->`Setting`->`Plugins`搜索安装即可  
<img src="https://s1.ax1x.com/2020/09/16/wcTUMQ.png" width="100%">

* **使用**  
1. 安装并重启idea后，操作提交时会有此按钮  
<img src="https://s1.ax1x.com/2020/09/16/wcT2M4.png" width="100%">
 
2. 点击弹出  
<img src="https://s1.ax1x.com/2020/09/16/wcTHzD.png" width="100%">
 
3. 按需填写并确认后生成以下格式：  
<img src="https://s1.ax1x.com/2020/09/16/wcTjeA.png" width="100%">


## 利用conventional-changelog生成Change log
  * **安装**
  ```shell
  npm install -g conventional-changelog-cli
  ```
  * **生成**
  ```shell
  conventional-changelog -p angular -i CHANGELOG.md -s
  ```
  * **生成的md效果如下**  
  
  
  <img src="https://s1.ax1x.com/2020/09/16/wc7NY6.png" width="100%">



## 更多可参考:  
[Git 提交的正确姿势：Commit message 编写指南](https://www.cnblogs.com/daysme/p/7722474.html)  
[IDEA 中 Git Commit message 编写](https://blog.csdn.net/itguangit/article/details/99590995#t2)  
[Commit message 和 Change log 编写指南](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)  
