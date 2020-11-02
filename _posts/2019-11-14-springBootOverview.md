---
title: Spring Boot 简介
---
## Spring boot运行原理
Spring boot用一个总注解`@SpringBootApplication`开启自动配置，其核心注解是`@EnableAutoConfiguration`。这个注解利用了`@  Import`来导入配置。其基本原理是EnableAutoConfigurationImportSelector使用SpringFactoriesLoader.loadFactoryNames()这个方法扫描具有META-INFO/spring.factories文件的包。这个文件里，用名为`org.springframework.boot.autoconfigure.EnableAutoConfiguration`的配置项声明了包含的自动配置。

![autoConfigScreanCapture](/images/20191117AutoConfigScreanCapture.jpg)

上述配置项中的任一一个AutoConfiguration文件，一般都有条件注解。比如上述截图中的AopAutoConfiguration类。todo wujunpeng 好好看看条件注解相关的代码。

todo wujunpeng 根据书中6.5.4自己编写一个自动配置的starter pom。学习maven的打包，发布到仓库。
