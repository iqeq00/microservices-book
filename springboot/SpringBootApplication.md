# @SpringBootApplication

**Spring Boot 最核心的注解**，整个 Spring Boot 和 Spring Cloud 完整的微服务技术体系，本质上就是围绕着一系列的注解展开的。注解在 Spring 微服务的技术栈中，都扮演着举足轻重的角色和作用。注解的数量非常多，但是 Spring 体系把很多注解都归结到某一个，或者是某几个注解上。使得我们可以更简单的方式去使用注解。

## 修改配置信息

对于 Spring Boot 应用来说，一些配置信息都可以通过以下两种方式改变。这两种方式，没有哪个好，哪个不好。

Spring Boot 提倡约定优于配置，所以配置信息大量减少，都使用的默认配置。

### 配置文件方式

properties、yml 配置文件。

### 编码方式

尝试通过编码方式修改默认的 banner 信息显示。

## @SpringBootApplication

所在包：org.springframework.boot.autoconfigure

```java
package org.springframework.boot.autoconfigure;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.springframework.boot.SpringBootConfiguration;
import org.springframework.boot.context.TypeExcludeFilter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.ComponentScan.Filter;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;
import org.springframework.core.annotation.AliasFor;

/**
 * Indicates a {@link Configuration configuration} class that declares one or more
 * {@link Bean @Bean} methods and also triggers {@link EnableAutoConfiguration
 * auto-configuration} and {@link ComponentScan component scanning}. This is a convenience
 * annotation that is equivalent to declaring {@code @Configuration},
 * {@code @EnableAutoConfiguration} and {@code @ComponentScan}.
 *
 * @author Phillip Webb
 * @author Stephane Nicoll
 * @since 1.2.0
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM,
				classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication
```

说明：标识着一个 configuration 的 class（@SpringBootConfiguration 这个注解起作用），声明了一个或者多个 @Bean 方法，并会出发自动装配（@EnableAutoConfiguration 这个注解起作用）和组件扫描（@ComponentScan 这个注解起作用）。也就是说 @SpringBootApplication 这个注解是@SpringBootConfiguration、@EnableAutoConfiguration、@ComponentScan 这三个注解的集合，包含这三个注解的全部功能，目的是更加便捷的使用注解。

**@Target、@Retention、@Documented、@Inherited 这四个注解，是 jdk 自带的。**

