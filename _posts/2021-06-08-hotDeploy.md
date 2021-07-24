---
title: SpringBoot 热部署
---

## 概述
最近由于团队缺人，做了一个前后端不分离的web项目。因为看过那本《JavaEE开发的颠覆者: Spring Boot实战》，之前在当当也写过一些前端代码，所以理所当然的选了AngularJs1 + BootStrap + SpringBoot作为技术栈来完成任务。说实在的，SpringBoot基本的功能，书里写的还是挺详细的，只是AngularJs和BootStrap如果没有人指导一下，上手还是挺困难的。说起这事儿，还要感谢当时在当当的时候那个带我的小师父。有问必答不厌其烦，且能引导人思考，挺好的人。

写前端代码的时候，有时候想立刻得到调整后的结果，如果仅仅是样式，可以通过浏览器控制台调试。但是涉及到js脚本，除了热部署我还真没想到其他更好的方式，搜了很多网页，最后记录一下亲测好用的方法。

## 步骤
### 1 导包
``` xml
<!--SpringBoot热部署配置 -->
<dependency> 
  <groupId>org.springframework.boot</groupId> 
  <artifactId>spring-boot-devtools</artifactId> 
  <scope>runtime</scope> 
  <optional>true</optional> 
  <version>2.4.5</version> 
</dependency>
```
### 2 配置项  
在配置项中增加这一行：`spring.devtools.restart.enabled=true`
### 3 IDEA配置  
#### 3.1 第一处修改
在这个路径下：`Preferences` -> `Build, Execution, Deployment` -> `Complier`，找到`Build project automatically`，选中。例子图示如下：  
![hotDeployConfig1](/images/20210608hotdeployconfig1.png)
#### 3.2 第二处修改
Mac系统使用组合键：`shift + option + command + /`，弹窗中单击`Registry`，新弹窗勾选`compiler.automake.allow.when.app.running`  
![hotDeployConfig2](/images/20210608hotdeployconfig2.png)

![hotDeployConfig3](/images/20210608hotdeployconfig3.png)
