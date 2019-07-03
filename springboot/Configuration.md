# @Configuration

所在包：org.springframework.context.annotation

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

标识一个 java 类声明了一个或者多个 @Bean 方法，而且可以被 Spring 的容器进行处理，生成 Bean 的定义，以及针对于运行期的 Bean 服务请求。

例如：

```java
@Configuration
   public class AppConfig {
  
       @Bean
       public MyBean myBean() {
           // instantiate, configure and return bean ...
       }
   }
```

@Configuration 类通常是要么通过 AnnotationConfigApplicationContext ，要么通过 AnnotationConfigWebApplicationContext 来启动的。

针对之前的代码的一个简单实例：

```java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
   ctx.register(AppConfig.class);
   ctx.refresh();
   MyBean myBean = ctx.getBean(MyBean.class);
   // use myBean ...
```

首先创造一个 AnnotationConfigApplicationContext 应用上下文的实例，接着把 Java Config 配置类注册到应用上下文当中，刷新应用上下文以后就可以通过 getBean 方法获取之前定义好的 MyBean 实例，根据后面的请求，进行 MyBean 操作。

**这也解释了，我们在 Spring Boot 应用当中，配置的所有 Java 配置类，怎么就能被 Spring Boot 或者 Spring 找到的。**