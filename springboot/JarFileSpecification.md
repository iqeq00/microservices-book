# Jar 文件规范

为了更好的理解后续内容，先了解下 Jar 文件规范。

## Main-Class 位置

对于一个标准的 java jar 归档文件，规定一旦指定了 Main-Class 类信息，那么这个类连同它的包结构就必须放在这个 jar 归档文件的最顶层目录下，相当于整个应用工程的代码都放在了 jar 文件的最顶层目录下。这个类不允许嵌套，如果出现了嵌套（jar 文件内嵌套了 jar文件内的 Main-Class 类），会导致 Main-Class 类不能执行，这个类只有在 jar 归档文件的最顶层目录下才能被执行。

比如：Spring Boot 打包后的 jar。

![MANIFEST.MF-01](../images/MANIFEST.MF-01.jpg)

**这里指定了 Main-Class 类信息，那么 JarLauncher 这个入口类连同它的包结构 org.springframework.boot.loader 就必须在 Spring Boot 打包后的 jar 的最顶层目录下。** 



![SpringBootJarDir](../images/SpringBootJarDir.jpg)

这就明白了，为什么 Spring Boot 不把 spring-boot-loader.jar 这个依赖传递给实际的 Spring Boot 应用（这种就是规范里说的不允许出现 jar 文件嵌套），而是采用把这个依赖的**内容**打包进实际的 Spring Boot 应用。

## Jar 文件嵌套

标准规范里 jar 文件是不允许嵌套的，不能在一个 jar 文件内嵌套另外一个 jar 文件。Java 类加载器是加载不了嵌套 jar 文件的情况，但是有时为了合理的规避规范问题，有两种做法：

### 传统方式

通过 maven 的一些打包插件，比如：maven-shade-plugin 等相关插件。

假如一个应用工程依赖了 10 个三方依赖 jar，maven-shade-plugin 为了迎合 jar 文件规范。除了把应用工程自身的 class 文件打包到应用工程 jar 文件里，还把这 10 个三方依赖 jar 进行解压缩，把三方依赖 jar 的所有 class 文件同时打包到应用工程 jar 文件里。这样的话，应用工程 jar 文件里就没有嵌套其他 jar 文件，就符合了 jar 文件规范。

虽然解决了规范问题，但是也包含几个问题：

- 文件混乱

  应用工程自身的 class 文件和三方依赖 jar 里的 class 文件都放在了一个 jar 内，会导致打包后 jar 内的文件非常的混乱。

- 文件覆盖

  这 10 个三方依赖 jar 里的文件，出现了文件同名，比如：配置文件名字、java 类名、package 包名、handlers 文件名、schemas 文件名等同名情况。

  那么就会产生文件覆盖问题，这就会导致应用无法生产使用。

**遗留问题：maven-assembly-plugin 这些插件是不是也可以做到把三方 jar 放在一个指定的目录来打包？**

## 不寻常的 Spring Boot

通过上面的内容，发现 Spring Boot 实际打包出来的 jar 是不符合规范的。应用类加载器只会加载符合 jar 文件规范的类或 jar 文件，只有  spring-boot-loader.jar 解压后的所有类可以被加载。

### 源码包路径

Spring Boot 只是把 spring-boot-loader.jar 这个依赖的包内容解压后，放在应用工程 jar 的最顶层，而应用工程自身的 class 文件却放在了 BOOT-INF/classes 内，这种目录结构内的类是不能被应用类加载器加载的。

### 依赖包路径

Spring Boot 还把三方依赖 jar 放在 BOOT-INF/lib 内，这就产生了嵌套问题，jar 文件内嵌套了其他三方依赖 jar。这种目录结构内的 jar 包是不能被类应用加载器加载的。

### 解决办法

Spring Boot 创建了一个自定义的 Spring Boot Loader 类加载器，来加载 BOOT-INF/classes 目录下的类文件和 BOOT-INF/lib 目录下的 jar 文件。

## FatJar

类似于 Spring Boot 这种 jar 打包方式，jar 文件内嵌套了其他的 jar 文件，称为 FatJar。

