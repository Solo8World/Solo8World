---
title: GIT Flow流程管理规范
date: 2020-09-10 16:59:33
tags:
- GIT
- 开发规范
categories:
- 技术
toc: true
description: Git Flow Integration插件是基于git flow(开发流程管理模型)的idea插件。 
---


## **简介**
>Git Flow Integration插件是基于git flow(开发流程管理模型)的idea插件。  

git flow开发流程管理模型主要使用五类分支：  
   * master 主分支:生产环境部署分支, master分支永远与生产环境部署的版本保持同步。  
   * develop 开发分支:永远是下一个版本中已开发完成的新特性的最新代码。 
   * feature/功能名称: 新功能开发分支,为具体某一新功能开发而存在的临时分支,开发完成后合并回develop分支,并删除。 
   * release/版本号:预上线分支,版本中所有功能开发完毕并合并到develop以后,从develop开出release分支进行整体测试，测试稳定后打上tag同时并入develop、master分支。  
   * hotfix/bug:生产环境bug修复分支,修复后双向合并到develop和master。   

## **安装**  
  * Git Flow Integration依赖于本地git flow，所以先安装git flow软件包:  
    > 执行`git flow version` 检查是否已安装
    * Ubuntu or Debian安装
    ```shell
    apt-get install git-flow 
    ```
    * Archlinux 安装
    ```bash
    yaourt -S gitflow-avh 
    ```
    * Windows安装  
    请参考 [Windows下gitflow的安装方法](https://juejin.im/post/6844903682027307022)  
  * 安装Idea插件  
   idea中`File`->`Setting`->`Plugins`搜索安装即可  
   <img src="https://s1.ax1x.com/2020/09/16/wc7rmd.png" width="100%">  
   

## **插件使用**
1. 安装并重启idea后,右下角出现`No Gitflow`  
     <img src="https://s1.ax1x.com/2020/09/16/wc7Imj.png" width="100%">
2. 点击init repo进行初始化  
 _注：git flow init之前请确保本地没有尚未提交的更改_  
<img src="https://s1.ax1x.com/2020/09/16/wc7jcF.png" width="100%">

3. 确认后git flow自动基于master创建并切换到develop分支，并在右下角gitflow中出现操作选项    
<img src="https://s1.ax1x.com/2020/09/16/wcHp7R.png" width="100%">

    ```
    * 新功能开发分支线（feature）
    * 开发版本bug修复线（bugfix）
    * 新版本发布线/预上线（release）
    * 生产版本bug修复线（hotfix）
    ```
4. 开始使用，开发系统管理模块  
<img src="https://s1.ax1x.com/2020/09/16/wcHVje.png" width="100%">

5. 确认后自动生成feature为前缀的开发分支，并提供以下操作  
<img src="https://s1.ax1x.com/2020/09/16/wcHQ4P.jpg" width="70%">

`Finish feature` 结束分支：合并当前开发分支至develop并删除当前开发分支  
`Publish feature`推送分支至远程仓库

## 扩:分支管理详细  
 * master
      * 主分支, 随项目一直存在的长期分支.
      * master分支HEAD所在的位置, 永远是当前生产环境的代码.
      * master分支不允许直接提交代码, 仅允许从release或者hotfix分支通过merge request合并代码.
 * develop
      * 开发分支, 随项目一直存在的长期分支.
      * develop分支的HEAD所在的位置, 永远是下一个版本中已开发完成的新特性的最新代码.
      * develop分支的代码每天自动构建并部署到测试环境.
      * develop分支不允许直接提交代码, 仅允许从feature, release或者hotfix分支通过merge request合并代码.
      * 当develop分支中下一个版本的新特性已经全部开发完毕后, 从develop分支开出release分支, 进入测试阶段.
      * 在下个版本的release分支创建之前, 非下个版本的feature分支不允许向develop分支合并.
 * feature分支
      * feature分支是一类以feature/为前缀(gitflow默认值, 可以更换)的分支的统称.
      * 每一个feature分支从develop分支新建, 进行**某一个功能**的开发. 功能开发并测试稳定后, feature分支将合并回develop分支.
      * 同一个人可以同时开发多个feature分支, 同一个feature分支也可以同时被多个人开发.
      * 多个feature同时开发的情形, 后开发完的分支在最后合并回develop时, 往往会遇到冲突的情况. 此时一般遵循一下两种方法解决冲突.
         * 先将最新的develop分支向当前feature分支进行合并, 然后再将当前feature分支合并回develop.
         * 先将当前的feature分支向最新的develop分支进行rebase, 然后再将当前feature分支合并回develop.
         * 两种方式的比较, 可以参考文章Merging vs. Rebasing, 或其中文版代码合并：Merge、Rebase 的选择
 * release分支
      * release分支是一类以release/为前缀(gitflow默认值, 可以更换)的分支的统称.
      * develop分支上的下一个版本的所有新特性开发完毕, 从develop分支开出一个该版本的release分支, 并进行测试.
      * release分支不允许进行新特性开发, 而只进行bug修复和更新版本mata信息(如版本号, 构建日期, 更新日志等), 并且可以不定期将新的bug修复改动合并回develop.
      * 当release充分测试稳定后, 同时合并进入master分支和develop分支, 并在master分支上的建议该release版本的TAG.
 * hotfix分支
     * 当生产环境发现紧急bug时，可以通过新建hotfix分支，来修复bug, 修复后双向合并到develop和master.  

## 更多可参考:  
[git-flow 的工作流程| Learn Version Control with Git](https://www.git-tower.com/learn/git/ebook/cn/command-line/advanced-topics/git-flow)  
[Git 工作流程- 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2015/12/git-workflow.html)  
[git flow使用规范](https://zhuanlan.zhihu.com/p/153459671)
