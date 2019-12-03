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

## 使用环境 API

外部的值，可以通过把  org.springframework.core.env.Environment 组件注入到 @Configuration 所在的类当中，从而使用它们。

比如使用自动装配注解来注入：

### @Autowired 

自动装配注解，自动装配到所需的类里面。

```java
   @Configuration
   public class AppConfig {
  
       @Autowired 
       Environment env;
  
       @Bean
       public MyBean myBean() {
           MyBean myBean = new MyBean();
           myBean.setName(env.getProperty("bean.name"));
           return myBean;
       }
   }
```

这里的 env 被 @Autowired 自动装配到 AppConfig 类里，而 myBean 的 name 值就是通过 `env.getProperty("bean.name")` 的方式来获取的外部值。

### @PropertySource

属性源注解。通过环境对象（Environment）去解析的属性位于一个或者多个 属性源（property source）对象当中，而且被 @Configuration 修饰的类还可以通过使用 @PropertySource 注解贡献属性源（property source）到环境对象（Environment）中。

也就是说，只要通过 @PropertySource 指定了外部文件，那么就可以使用外部文件里面的值了。

```java
   @Configuration
   @PropertySource("classpath:/com/acme/app.properties")
   public class AppConfig {
  
       @Inject 
       Environment env;
  
       @Bean
       public MyBean myBean() {
           return new MyBean(env.getProperty("bean.name"));
       }
   }
```

例如上面的 `bean.name` 就是定义在 classpath:/com/acme/app.properties 外部文件里面的值。

### @Value

值注解。外部的值可以通过值注解注入到 @Configuration 修饰的类中。

```java
   @Configuration
   @PropertySource("classpath:/com/acme/app.properties")
   public class AppConfig {
  
       @Value("${bean.name}") 
       String beanName;
  
       @Bean
       public MyBean myBean() {
           return new MyBean(beanName);
       }
   }
```

当实例化或者加载 AppConfig 类实例的时候，就会从 app.properties 外部配置当中，找到 bean.name 的配置信息，并把值设置给 beanName。

这种方式经常会与 Spring 的 PropertySourcesPlaceholderConfigurer 搭配使用。

## @Import

导入注解。可以通过它来组合两个或者多个被 @Configuration 修饰的类，把多个配置类组合到一起，目的是实现配置的拆分。

类似于 Spring XML 配置方式中的 `<import>` 标签。

由于 @Configuration 对象，是做为一个 Spring Bean 对象在容器当中被管理的，因此被导入的 @Configuration 对象（配置）也是可以被注入的，比如通过构造器来注入。

```java
   @Configuration
   public class DatabaseConfig {
  
       @Bean
       public DataSource dataSource() {
           // instantiate, configure and return DataSource
       }
   }
  
   @Configuration
   @Import(DatabaseConfig.class)
   public class AppConfig {
  
       private final DatabaseConfig dataConfig;
  
       public AppConfig(DatabaseConfig dataConfig) {
           this.dataConfig = dataConfig;
       }
  
       @Bean
       public MyBean myBean() {
           // reference the dataSource() bean method
           return new MyBean(dataConfig.dataSource());
       }
   }
```

DatabaseConfig 做为 AppConfig 的一个变量，通过构造方法注入进来，然后就可以使用 DatabaseConfig 相关的配置信息了。

### 启动

现在，不管是 AppConfig，还是被导入的 DatabaseConfig 都可以通过注册到 Spring Context 当中来启动。只需要注册 AppConfig 就可以了，因为 DatabaseConfig 已经被包含在 AppConfig 中了。

启动方法：给 Spring Context 注册导入的 @Configuration 类。

```java
new AnnotationConfigApplicationContext(AppConfig.class);
```

## @Profile

多环境注解。实际的项目当中，至少分为开发、测试、生产三种环境，每种环境的配置信息，应该都是不一样的，比如数据库、缓存等信息。

@Configuration 所修饰的类还可以被 @Profile 修饰，来指定不同环境信息。

```java
   @Profile("development")
   @Configuration
   public class EmbeddedDatabaseConfig {
  
       @Bean
       public DataSource dataSource() {
           // instantiate, configure and return embedded DataSource
       }
   }
  
   @Profile("production")
   @Configuration
   public class ProductionDatabaseConfig {
  
       @Bean
       public DataSource dataSource() {
           // instantiate, configure and return production DataSource
       }
   }
```

上面的例子分别声明了开发阶段和生产阶段的数据源配置。

在 Spring Bean 的方法层次上，还可以声明一些 @Profile 的条件。

```java
   @Configuration
   public class ProfileDatabaseConfig {
  
       @Bean("dataSource")
       @Profile("development")
       public DataSource embeddedDatabase() { ... }
  
       @Bean("dataSource")
       @Profile("production")
       public DataSource productionDatabase() { ... }
   }
```

都是叫做 dataSource 的 Spring Bean，配置信息也在同一个 @Configuration 类当中，只是通过 @Profile 的值不同来区分。但是这是一个条件判断，当前环境如果是 development，DataSource Spring Bean 就是 embeddedDatabase，而当前环境如果是 production，DataSource Spring Bean 就是 productionDatabase。

上下两种方式，本质上都是一样的，只是上面的配置方式需要两个 @Configuration 配置类，而下面的方式，一个 @Configuration 配置类就可以了。

## @ImportResource

导入配置文件注解。@Configuration 类也可以使用 @ImportResource 注解将 Spring XML 配置文件导入 @Configuration类。

```java
   @Configuration
   @ImportResource("classpath:/com/acme/database-config.xml")
   public class AppConfig {
  
       @Inject DataSource dataSource; // from XML
  
       @Bean
       public MyBean myBean() {
           // inject the XML-defined dataSource bean
           return new MyBean(this.dataSource);
       }
   }
```

database-config.xml 配置文件中的内容就被导入到 AppConfig 当中了。

用途不是特别大，不过对于老系统的架构迁移来说，也是一种解决方案，比如从 Spring XML 架构迁移到 Spring Boot 架构。

## 嵌套

@Configuration 类可以相互嵌套。

```java
   @Configuration
   public class AppConfig {
  
       @Inject 
       DataSource dataSource;
  
       @Bean
       public MyBean myBean() {
           return new MyBean(dataSource);
       }
  
       @Configuration
       static class DatabaseConfig {
           @Bean
           DataSource dataSource() {
               return new EmbeddedDatabaseBuilder().build();
           }
       }
   }
```

内层的 DataSource 也会自动注入到外层的 dataSource 当中。

在这种情况下，仅需要针对应用程序上下文注册 AppConfig。由于是嵌套的 @Configuration 类，因此将自动注册 DatabaseConfig。当 AppConfig 和 DatabaseConfig 之间的关系已经很明显的时候，就无需使用 @Import 注解了。

此外，嵌套的 @Configuration 类可以与 @Profile 注解一起搭配使用，为内层的 @Configuration 类提供同一配置 bean 的两个不同选择，比如开发环境和测试环境，相同 bean 的两种选择。

## 结尾

在一个 Spring Boot 的项目中，基本都会存在一个 config 的代码包。里面存放着项目的各种各样配置类，而配置的具体值，很可能放置在外部的 properties 或者 yml 文件中。