# Spring Boot 应用配置分析

## UTC

一种时间格式，国际化的项目，最好采用 utc 格式，因为 utc 的格式是没有时区的问题，在日期传递过程中效果是最好的。

## Banner

控制台会打印 Spring Boot Logo 以及版本信息，这个操作可以通过配置实现自定义。

[例如](https://www.jianshu.com/p/a53f324c92f2)

## 进程运行

从控制台的输出信息可以知道 Spring Boot 都是运行在进程内的，如何查看？

指令：`lsof -i:8080` 或 `ps -ef | grep java`

## Profile

**一般模式**：一套配置文件，基于开发、测试、线上等多种环境时，构建多个整套配置文件，在项目启动的时候，通过一个参数来实现配置的切换。目的解决代码和配置分离，但不是最好的方式。

**比较好的方式：代码和配置完全隔离**。项目工程当中是看不到任何配置信息的，配置信息是集中的放在 Spring Cloud Config 中，可以对其加密。

**更好的方式：配置中心**。比如：携程的 [Apollo](https://github.com/ctripcorp/apollo)。

## Gradle

推荐学习。

## 启动类

每个 Spring Boot 应用都有一个启动类。

启动类需要以下几个条件：

- 由 @SpringBootApplication 注解修饰

- 启动类当中有一个 main 方法，也只有一句启动代码

  ```java
  SpringApplication.run(xxx.class, args);
  ```

## 配置文件

Spring Boot 提供了两种类型的配置文件形式，但是都是遵循约定优于配置，都叫做 application 这个名字，虽然可以，尽量不要修改这个名字。

### properties 文件格式

传统的经典文件格式。

**application.properties**

```properties
server.port=8080
server.address=127.0.0.1
server.context-path=/
```

### yml 文件格式

[YAML](https://zh.wikipedia.org/wiki/YAML)（Yet Another Markup Language），一种新的文件格式。

**application.yml**

```yaml
server:
	port: 8080
	address: 127.0.0.1
	context-path: /
```

感觉上 yml 的方式要更简洁一点，可以少写前缀 server 很多次。

**需要注意一点：**当 yml 配置的最后一层，也就是实际值和前面对应的 key 后面的冒号之间，有一个空格。

例如：port: 8080，冒号和 8080 之间必须有空格，可以观察 idea 或者 md 的引用代码区域是否正常**高亮显示**来判断书写正确与否。