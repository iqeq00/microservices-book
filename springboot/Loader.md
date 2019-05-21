---
typora-copy-images-to: ..\images
---

# Spring Boot Loader 源码分析

## Spring Boot jar

我们之间就发现 Spring Boot 打包出来的 jar 解压后，有三个目录。

![SpringBootJarDir](../images/SpringBootJarDir.jpg)

### BOOT-INF

#### classes

当前工程编译好的结果文件，包含 src/main/java 和 src/main/resources 下的所有文件。

#### lib

当前工程依赖的所有 jar 文件（第三方 jar 包）。

### META-INF

只有一个文件：MANIFEST.MF。

![MANIFEST.MF-01](../images/MANIFEST.MF-01.jpg)

### org

由 Spring Boot 提供的一堆字节码文件，入口类就在这里。

```java
org.springframework.boot.loader.JarLauncher
```

## org package 是怎么来的

BOOT-INF 是通过编译工程文件，META-INF 是打包时自动生成，那么 org package 是怎么来的？

```groovy
plugins {
   id 'org.springframework.boot' version '2.1.4.RELEASE'
   id 'java'
}
```

还记得我们之间配置的 gradle plugin 么？既然打包是由 org.springframework.boot 这个插件完成，那么答案肯定在这里。

根据 org 的目录结构，尝试增加一个依赖到工程里。

org.springframework.boot:spring-boot-loader （**在开发阶段，因为有 gradle 插件的存在，原则上这个依赖是不引入的，这里是研究需要**）

```groovy
dependencies {
   implementation 'org.springframework.boot:spring-boot-starter-web'
   implementation 'org.springframework.boot:spring-boot-loader'
   testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

再观察 spring-boot-loader.jar，发现和上面的 org package 一模一样。

![spring-boot-loader](../images/spring-boot-loader.jpg)

结论：说明 Spring Boot Gradle Plugin 在用 bootJar 任务对 Spring Boot 工程进行打包的时候，是把 spring-boot-loader.jar 这个 jar 解压，随后把解压内容放置在 Spring Boot 的应用 jar 里。而spring-boot-loader.jar 这个依赖，肯定是在 Spring Boot Gradle Plugin 里被依赖的。

疑问：为什么不直接把这个依赖传递给实际的 Spring Boot 应用，而是采用这么麻烦的方式，把这个依赖的**内容**打包进实际的 Spring Boot 应用。

## JarLauncher 启动类

根据 MANIFEST.MF 文件的描述。

Spring Boot 的启动类就是 

```java
Main-Class: org.springframework.boot.loader.JarLauncher
```

而 Spring Boot 的应用主类是

```java
Start-Class: com.lichee.microservices.MicroservicesApplication
```

### 源码

```java
package org.springframework.boot.loader;

import org.springframework.boot.loader.archive.Archive;

/**
 * {@link Launcher} for JAR based archives. This launcher assumes that dependency jars are
 * included inside a {@code /BOOT-INF/lib} directory and that application classes are
 * included inside a {@code /BOOT-INF/classes} directory.
 *
 * @author Phillip Webb
 * @author Andy Wilkinson
 */
public class JarLauncher extends ExecutableArchiveLauncher {

   static final String BOOT_INF_CLASSES = "BOOT-INF/classes/";

   static final String BOOT_INF_LIB = "BOOT-INF/lib/";

   public JarLauncher() {
   }

   protected JarLauncher(Archive archive) {
      super(archive);
   }

   @Override
   protected boolean isNestedArchive(Archive.Entry entry) {
      if (entry.isDirectory()) {
         return entry.getName().equals(BOOT_INF_CLASSES);
      }
      return entry.getName().startsWith(BOOT_INF_LIB);
   }

   public static void main(String[] args) throws Exception {
      new JarLauncher().launch(args);
   }

}
```

### 类关系图

![JarLauncherClass](../images/JarLauncherClass.jpg)

从图中可以看出来，最顶层的启动器是 Launcher，其次是 ExecutableArchiveLauncher。最终的实现分别两种模式，一个是 JarLauncher（针对于 jar 归档文件），另一个是 WarLauncher（针对于 war 归档文件）。从这里也就说明了为什么 Spring Boot 的应用可以通过 jar 和 war 两种方式运行。

### 源码分析

通过代码可以发现，实际程序的入口是在：

```java
new JarLauncher().launch(args);
```

实际调用的 launch 方法是 Launcher.java 里的 launch 方法。

```java
/**
 * Launch the application. This method is the initial entry point that should be
 * called by a subclass {@code public static void main(String[] args)} method.
 * @param args the incoming arguments
 * @throws Exception if the application fails to launch
 */
protected void launch(String[] args) throws Exception {
   JarFile.registerUrlProtocolHandler();
   ClassLoader classLoader = createClassLoader(getClassPathArchives());
   launch(args, getMainClass(), classLoader);
}
```

这里最重要的就是 ClassLoader 这行代码，类加载器。

#### getClassPathArchives

返回所有符合条件的 jar 或 工程文件，并包装成一个类型为 Archive 的 List 对象。这里的符合条件是指在 BOOT-INF/lib/ 下的 jar 文件，或在 BOOT-INF/classes/ 下的所有工程文件，用来构建 classpath。

```java
@Override
protected List<Archive> getClassPathArchives() throws Exception {
   List<Archive> archives = new ArrayList<>(
         this.archive.getNestedArchives(this::isNestedArchive));
   postProcessClassPathArchives(archives);
   return archives;
}
```
定位当前执行的具体 jar 文件或者文件目录，通过磁盘上的绝对路径来定位，返回一个 Archive 对象。

```java
protected final Archive createArchive() throws Exception {
   ProtectionDomain protectionDomain = getClass().getProtectionDomain();
   CodeSource codeSource = protectionDomain.getCodeSource();
   URI location = (codeSource != null) ? codeSource.getLocation().toURI() : null;
   String path = (location != null) ? location.getSchemeSpecificPart() : null;
   if (path == null) {
      throw new IllegalStateException("Unable to determine code source archive");
   }
   File root = new File(path);
   if (!root.exists()) {
      throw new IllegalStateException(
            "Unable to determine code source archive from " + root);
   }
   return (root.isDirectory() ? new ExplodedArchive(root)
         : new JarFileArchive(root));
}
```

返回与指定过滤器（EntryFilter）所匹配的嵌套归档文件。

```java
@Override
public List<Archive> getNestedArchives(EntryFilter filter) throws IOException {
    List<Archive> nestedArchives = new ArrayList<>();
    for (Entry entry : this) {
        if (filter.matches(entry)) {
            nestedArchives.add(getNestedArchive(entry));
        }
    }
    return Collections.unmodifiableList(nestedArchives);
}
```
EntryFilter 过滤器，判断所指定的文件（具体 jar 文件或者文件目录）是否是一个嵌套条目 ，应该添加到classpath里，每个指定的文件都会调用一次。

条件：在 BOOT-INF/classes/ 下的工程文件或者在 BOOT-INF/lib/ 下的第三方 jar 包。

**注意：其实这里就是在判断，要执行的 jar 文件是否是按照 Spring Boot 特有的目录结构来放置工程文件，以及所依赖的第三方 jar 包。只有满足条件的工程文件或所依赖的第三方 jar 包才会进入下一步，也就是通过自定义类加载器来加载这些满足条件的文件，这里就需要 jar 文件规范相关知识。**

```java
static final String BOOT_INF_CLASSES = "BOOT-INF/classes/";
static final String BOOT_INF_LIB = "BOOT-INF/lib/";

@Override
protected boolean isNestedArchive(Archive.Entry entry) {
    if (entry.isDirectory()) {
        return entry.getName().equals(BOOT_INF_CLASSES);
    }
    return entry.getName().startsWith(BOOT_INF_LIB);
}
```

#### postProcessClassPathArchives

是一个空方法，事后处理方法，回调方法。

```java
/**
 * Called to post-process archive entries before they are used. Implementations can
 * add and remove entries.
 * @param archives the archives
 * @throws Exception if the post processing fails
 */
protected void postProcessClassPathArchives(List<Archive> archives) throws Exception {
}
```

#### createClassLoader

创建一个指定归档文件（Archive）的类加载器，也就是用来加载 getClassPathArchives 所返回的集合（jar 或者 工程文件），应用类加载器（也可以叫做系统类加载器）是加载不了这些文件的，就必须自己创建一个新的类加载器，用来加载这些存在于自定义目录内的文件。

这个方法是把返回的 List<Archive> 对象转撑一个 List<URL> 对象，URL 表示文件的绝对路径。

```java
**
 * Create a classloader for the specified archives.
 * @param archives the archives
 * @return the classloader
 * @throws Exception if the classloader cannot be created
 */
protected ClassLoader createClassLoader(List<Archive> archives) throws Exception {
   List<URL> urls = new ArrayList<>(archives.size());
   for (Archive archive : archives) {
      urls.add(archive.getUrl());
   }
   return createClassLoader(urls.toArray(new URL[0]));
}
```

#### LaunchedURLClassLoader

Spring Boot 的类加载器，urls 表示所有需要加载文件的 url（jar 文件的绝对路径），getClass().getClassLoader() 表示父加载器（也就是应用类加载器）。

**注意：在创建一个类加载器的时候，一定要指定它的父加载器 getClass().getClassLoader(），这这个父加载器其实就是应用类加载器。**

这个方法创建一个指定归档文件（URL）的类加载器。

```java
/**
 * Create a classloader for the specified URLs.
 * @param urls the URLs
 * @return the classloader
 * @throws Exception if the classloader cannot be created
 */
protected ClassLoader createClassLoader(URL[] urls) throws Exception {
   return new LaunchedURLClassLoader(urls, getClass().getClassLoader());
}
```

