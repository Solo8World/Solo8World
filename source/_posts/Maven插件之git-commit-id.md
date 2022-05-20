---
title: Maven插件之git-commit-id
date: 2020-09-05 17:00:58
tags:
- GIT
toc: true
categories:
- [技术,工具]
description: 让我们通过接口就晓得后台代码的版本。该插件会在源码编译打包时生成git版本信息、打包信息，便于后期运维。  
---

# Maven插件 `git-commit-id-plugin`
> 让我们通过接口就晓得后台代码的版本。    

该插件会在源码编译打包时生成git版本信息、打包信息，便于后期运维。

## [git-commit-id-plugin](https://github.com/git-commit-id/git-commit-id-maven-plugin) 使用
  * pom文件引入
   ```xml
    <plugin>
      <groupId>pl.project13.maven</groupId>
      <artifactId>git-commit-id-plugin</artifactId>
      <version>2.2.0</version>
      <configuration>
          <verbose>true</verbose>
          <generateGitPropertiesFile>true</generateGitPropertiesFile>
          <injectAllReactorProjects>true</injectAllReactorProjects>
      </configuration>
  </plugin>
   ```
  * 重新打包，jar包中自动生成`git.properties`文件，如下：  
<img src="https://s1.ax1x.com/2020/09/16/wcHBCV.png" width="100%">
  
  * 编写项目版本信息查询接口
  ```java
    @Resource
    private GitProperties gitProperties;

    @GetMapping("/version")
    HashMap<String, String> version() {
        HashMap<String, String> versionInfo = new HashMap<>(8);
        versionInfo.put("branch", gitProperties.getBranch());
        versionInfo.put("commitId", gitProperties.getCommitId());
        versionInfo.put("commitMessage", gitProperties.get("commit.message.full"));
        versionInfo.put("commitUser", gitProperties.get("commit.user.name"));
        versionInfo.put("commitTime", DateUtil.date(Long.parseLong(gitProperties.get("commit.time"))).toString());
        versionInfo.put("buildHost", gitProperties.get("build.host"));
        versionInfo.put("buildUser", gitProperties.get("build.user.name"));
        versionInfo.put("buildTime", DateUtil.date(Long.parseLong(gitProperties.get("build.time"))).toString());
        return  versionInfo;
    }
  ```
  * 请求接口  
<img src="https://s1.ax1x.com/2020/09/16/wcHRER.png" width="100%">
  
## 注意事项
* springBoot 2.3.0之后的版本无法直接注入GitProperties，
需要在启动类声明GitProperties类的Bean属性。

## 更多请参考  
[该插件的github](https://github.com/git-commit-id/git-commit-id-maven-plugin)
