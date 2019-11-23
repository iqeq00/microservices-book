# @Configuration

所在包：org.springframework.context.annotation

所在 jar：spring-context.jar（5.1.6.RELEASE）

用于进行配置，修饰类，标示某一个类是配置类。

## 源码

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {

	/**
	 * Explicitly specify the name of the Spring bean definition associated with the
	 * {@code @Configuration} class. If left unspecified (the common case), a bean
	 * name will be automatically generated.
	 * <p>The custom name applies only if the {@code @Configuration} class is picked
	 * up via component scanning or supplied directly to an
	 * {@link AnnotationConfigApplicationContext}. If the {@code @Configuration} class
	 * is registered as a traditional XML bean definition, the name/id of the bean
	 * element will take precedence.
	 * @return the explicit component name, if any (or empty String otherwise)
	 * @see org.springframework.beans.factory.support.DefaultBeanNameGenerator
	 */
	@AliasFor(annotation = Component.class)
	String value() default "";

}
```

标识一个 java 类声明了一个或者多个被 @Bean 注解标示的方法，而且可以被 Spring 的容器进行处理，生成 Bean 的定义，以及针对于运行期的 Bean 服务请求。

## Java Config 实现方式

```java
	 @Configuration
   public class AppConfig {
  
       @Bean
       public MyBean myBean() {
           // instantiate, configure and return bean ...
       }
   }
```

@Configuration 类通常是要么通过 AnnotationConfigApplicationContext ，要么通过 AnnotationConfigWebApplicationContext 来启动、实现的。

针对之前的代码的一个简单实例：

```java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
   ctx.register(AppConfig.class);
   ctx.refresh();
   MyBean myBean = ctx.getBean(MyBean.class);
   // use myBean ...
```

首先创造一个 AnnotationConfigApplicationContext 应用上下文的实例，接着把 Java Config 配置类注册到应用上下文当中，刷新应用上下文以后就可以通过 getBean 方法获取（从 Spring 容器内）之前定义好的 MyBean 实例，根据后面的请求，进行 MyBean 操作。

**这也解释了，我们在 Spring Boot 应用当中，所有的 Java 配置类，怎么就能被 Spring Boot 或者 Spring 找到的。**

## Xml 实现方式

```xml
<beans>
    <context:annotation-config/>
    <bean class="com.acme.AppConfig"/>
</beans>
```

传统方式，在 Java Config 兴起以前，当时的标准就是使用 xml 来实现配置。在 Spring Boot 当中也是支持 xml 方式的，但是很少会这么用，通常都是使用注解方式。

```xml
<context:annotation-config/>
```

xml 方式必须要加上上面的命名空间，才能起效果。

## @ComponentScan 

组件扫描注解，修饰 @SpringBootApplication 注解。这个注解，主要是扫描应用的所有组件。

对于 Spring Boot 应用来说，常常会把包含 main 方法的这个类，也就是启动类，定义在一个包的最顶层。其他的控制器、服务层、数据层、组件层都定义在启动类的子包里。

因为根据注解扫描原则，注解会扫描这个包本身以及这个包的所有子包。所以一般 Spring Boot 的启动类都是位于整个应用的最顶层，应用的其他文件都在它的子包内。

