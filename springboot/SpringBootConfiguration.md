# @SpringBootConfiguration

所在包：org.springframework.boot

```java
package org.springframework.boot;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.springframework.context.annotation.Configuration;

/**
 * Indicates that a class provides Spring Boot application
 * {@link Configuration @Configuration}. Can be used as an alternative to the Spring's
 * standard {@code @Configuration} annotation so that configuration can be found
 * automatically (for example in tests).
 * <p>
 * Application should only ever include <em>one</em> {@code @SpringBootConfiguration} and
 * most idiomatic Spring Boot applications will inherit it from
 * {@code @SpringBootApplication}.
 *
 * @author Phillip Webb
 * @since 1.4.0
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {

}
```

标识一个提供 Spring Boot application 的 java 配置类，@Configuration 注解表示当前的 java 类是一个配置类。可以作为 Spring 标准 @Configuration 替换方案，配置可以自动被寻找到。

整个项目应用当中应该只包含一个 @SpringBootConfiguration 注解，大多数的 Spring Boot 应用都会直接从 @SpringBootApplication 注解当中继承 @SpringBootConfiguration 注解。

在整个项目应用当中，只使用一次，而且一般是通过使用 @SpringBootApplication 这个注解，达到间接使用 @SpringBootConfiguration 注解。