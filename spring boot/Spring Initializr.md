# Spring Boot 应用起步与配置

## 快速起步

[Spring Initializr](https://start.spring.io/)

通过勾选的方式，快速搭建一个项目的骨架，eclipse 和 idea 都有相关插件。

## Starters

**起步依赖**，Spring Boot 引入的理念，通过它可以很方便的把 Spring Boot 以及需要的第三方插件集成到应用当中。

## Version

因为有 starter 的存在，其实 Spring Boot 的版本号在 dependency 里是省略掉的，由 starter 统一管理，Spring Cloud 也是如此。

## bootRun

启动 Spring Boot 指令：gradle bootRun / gradlew bootRun。

也可以在 @SpringBootApplication 所注释的类里直接调用 run。

Spring Boot 启动是通过一个 main 方法，以 jar 包方式运行。web 服务器是以嵌入的方式存在于应用当中，所有的配置、三方依赖都在jar包内。其实 Tomcat 就是内嵌在 Spring Boot 应用内的（嵌入式服务器），这也是为什么 Tomcat 的一些配置可以通过 Spring Boot 的配置文件进行修改。

## 理念

约定优于配置

## 题外话

尝试下 Visual Studio Code，对于前端和后端的开发都有比较好的支持。