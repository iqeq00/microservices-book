# Spring Boot应用起步与配置

## 快速起步

[Spring Initializr](https://start.spring.io/)

通过勾选的方式，快速搭建一个项目的骨架，eclipse和idea都有相关插件。

## Starters

**起步依赖**，Spring Boot引入的理念，通过它可以很方便的把Spring Boot以及需要的第三方插件集成到应用当中。

## version

因为有starter的存在，其实Spring Boot的版本号在dependency里是省略掉的，由starter统一管理，Spring Cloud也是如此。

## bootRun

启动Spring Boot指令：./gradlew bootRun。

Spring Boot启动是通过一个main方法，以jar包方式运行。web服务器是以嵌入的方式存在于应用当中，所有的配置、三方依赖都在jar包内。

## 理念

约定优于配置