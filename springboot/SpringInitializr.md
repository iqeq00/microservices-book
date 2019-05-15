# Spring Boot 应用起步与配置

## 快速起步

[Spring Initializr](https://start.spring.io/)

通过勾选的方式，快速搭建一个项目的骨架，eclipse 和 idea 都有相关插件。

## Starters

**起步依赖**，Spring Boot 引入的理念，通过它可以很方便的把 Spring Boot 以及需要的第三方插件集成到应用当中。

## Version

因为有 starter 的存在，其实 Spring Boot 的版本号在 dependency 里是省略掉的，由 starter 统一管理，Spring Cloud 也是如此。

## 启动方式

Spring Boot 应用的启动方式一共有 4 种。

1. 在 @SpringBootApplication 所注释的类里直接调用 run 启动。

2. 通过 Spring Boot 启动指令

   ```groovy
   gradle bootRun 
   或者
   gradlew bootRun
   ```

3. 双击 Idea 里 Gradle 任务栏里面的 bootRun 任务。

4. 还可以找到这个应用打好的 jar 包，通过 Java 运行 jar 包的方式运行。

   ```java
    java -jar microservices-0.0.1-SNAPSHOT.jar 
   ```

**前三种是开发阶段常用的启动方式（建议使用第二、三种方式启动，第一种在工程有问题的时候，有时候也可以正常运行成功，例如：没有成功下载所有的 jar 包），最后一种是部署服务器时常用的启动方式。**

Spring Boot 启动是通过一个 main 方法，以 jar 包方式运行。web 服务器是以嵌入的方式存在于应用当中，所有的配置、三方依赖都在 jar 包内。其实 Tomcat 就是内嵌在 Spring Boot 应用内的（嵌入式服务器），这也是为什么 Tomcat 的一些配置可以通过 Spring Boot 的配置文件进行修改。

## 理念

约定优于配置

## 题外话

尝试下 Visual Studio Code，对于前端和后端的开发都有比较好的支持。