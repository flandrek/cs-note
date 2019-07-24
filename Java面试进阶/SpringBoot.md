## SpringBoot的优点

Spring Boot 优点非常多，如：

- 独立运行
- 简化配置
- 自动配置
- 无代码生成和XML配置
- 应用监控
- 上手容易



## SpringBoot 自动配置原理

Spring Boot的出现，得益于“习惯优于配置”的理念，没有繁琐的配置、难以集成的内容（大多数流行第三方技术都被集成），这是基于Spring 4.x提供的**按条件配置**Bean的能力。

### Spring Boot的配置文件

初识Spring Boot时我们就知道，Spring Boot有一个全局配置文件：application.properties 或 application.yml。

我们的各种属性都可以在这个文件中进行配置，最常配置的比如：server.port、logging.level.* 等等，然而我们实际用到的往往只是很少的一部分，那么这些属性是否有据可依呢？答案当然是肯定的，这些属性都可以在官方文档中查找到：

<https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#common-application-properties>

![img](E:\markdown笔记\图片\20181107113919249.png)

*（所以，话又说回来，找资料还得是官方文档，百度出来一大堆，还是稍显业余了一些）*

除了官方文档为我们提供了大量的属性解释，我们也可以使用IDE的相关提示功能，比如IDEA的自动提示，和Eclipse 的 YEdit 插件，都可以很好的对你需要配置的属性进行提示，下图是使用Eclipse的YEdit插件的效果，Eclipse的版本是：STS 4。

![img](E:\markdown笔记\图片\20181107114603751.png)

 以上，是Spring Boot的配置文件的大致使用方法，其实都是些题外话。

那么问题来了：**这些配置是如何在Spring Boot项目中生效的呢？**那么接下来，就需要聚焦本篇博客的主题：自动配置工作原理或者叫实现方式。

### 工作原理剖析

Spring Boot 关于自动配置的源码在 spring-boot-autoconfigure-x.x.x.x.jar 中：

![img](E:\markdown笔记\图片\20181107144449611.png)

当然，自动配置原理的相关描述，官方文档貌似是没有提及。不过我们不难猜出，Spring Boot的启动类上有一个@SpringBootApplication 注解，这个注解是 Spring Boot 项目必不可少的注解。那么自动配置原理一定和这个注解有着千丝万缕的联系！

### @EnableAutoConfiguration

![img](E:\markdown笔记\图片\20181107115331414.png)

@SpringBootApplication 是一个复合注解或派生注解，在 @SpringBootApplication 中有一个注解@EnableAutoConfiguration，翻译成人话就是**开启自动配置**，其定义如下：

![img](E:\markdown笔记\图片\20181107125035592.png)

 而这个注解也是一个派生注解，其中的关键功能由 @Import 提供，其导入的**AutoConfigurationImportSelector ** 的 selectImports( ) 方法通过**SpringFactoriesLoader.loadFactoryNames( )** 扫描所有具有 **META-INF/spring.factories** 的 jar 包。

spring-boot-autoconfigure-x.x.x.x.jar里就有一个这样的spring.factories文件。

这个 spring.factories 文件也是一组一组的 key = value 的形式，其中一个 key 是 EnableAutoConfiguration 类的全类名，而它的 value 是一个 xxxxAutoConfiguration 的类名的列表，这些类名以逗号分隔，如下图所示：

![img](E:\markdown笔记\图片\20181107130442565.png)

这个 @EnableAutoConfiguration 注解通过 @SpringBootApplication 被间接的标记在了 Spring Boot 的启动类上。在 SpringApplication.run(...) 的内部就会执行 selectImports() 方法，找到所有 JavaConfig 自动配置类的全限定名对应的 class，然后将所有自动配置类加载到 Spring 容器中。

### 自动配置生效

每一个 XxxxAutoConfiguration 自动配置类都是在某些条件之下才会生效的，这些条件的限制在 Spring Boot 中以注解的形式体现，常见的**条件注解**有如下几项：

> @ConditionalOnBean：当容器里有指定的bean的条件下。
>
> @ConditionalOnMissingBean：当容器里不存在指定bean的条件下。
>
> @ConditionalOnClass：当类路径下有指定类的条件下。
>
> @ConditionalOnMissingClass：当类路径下不存在指定类的条件下。
>
> @ConditionalOnProperty：指定的属性是否有指定的值，比如@ConditionalOnProperties(prefix=”xxx.xxx”, value=”enable”, matchIfMissing=true)，代表当xxx.xxx为enable时条件的布尔值为true，如果没有设置的情况下也为true。

以 ServletWebServerFactoryAutoConfiguration 配置类为例，解释一下全局配置文件中的属性如何生效，比如：server.port=8081，是如何生效的（当然不配置也会有默认值，这个默认值来自于org.apache.catalina.startup.Tomcat）。

![img](E:\markdown笔记\图片\20181107133910432.png)

在 ServletWebServerFactoryAutoConfiguration 类上，有一个 @EnableConfigurationProperties 注解：**开启配置属性**，而它后面的参数是一个 ServerProperties 类，这就是习惯优于配置的最终落地点。

![img](E:\markdown笔记\图片\20181107134400306.png)

在这个类上，我们看到了一个非常熟悉的注解：**@ConfigurationProperties**，它的作用就是从配置文件中绑定属性到对应的bean上，而 **@EnableConfigurationProperties** 负责导入这个已经绑定了属性的 bean 到 spring 容器中（见上面截图）。那么所有其他的和这个类相关的属性都可以在全局配置文件中定义，也就是说，真正“限制”我们可以在全局配置文件中配置哪些属性的类就是这些 **XxxxProperties** 类，它与配置文件中定义的 prefix 关键字开头的一组属性是唯一对应的。

至此，我们大致可以了解。在全局配置的属性如：server.port 等，通过 @ConfigurationProperties 注解，绑定到对应的 XxxxProperties 配置实体类上封装为一个bean，然后再通过 @EnableConfigurationProperties 注解导入到Spring容器中。

而诸多的 XxxxAutoConfiguration 自动配置类，就是 Spring 容器的 JavaConfig 形式，作用就是为 Spring 容器导入 bean，而所有导入的 bean 所需要的属性都通过 xxxxProperties 的 bean 来获得。

可能到目前为止还是有所疑惑，但面试的时候，其实远远不需要回答的这么具体，你只需要这样回答：

> Spring Boot启动的时候会通过@EnableAutoConfiguration注解找到META-INF/spring.factories配置文件中的所有自动配置类，并对其进行加载，而这些自动配置类都是以AutoConfiguration结尾来命名的，它实际上就是一个JavaConfig形式的Spring容器配置类，它能通过以Properties结尾命名的类中取得在全局配置文件中配置的属性如：server.port，而XxxxProperties类是通过@ConfigurationProperties注解与全局配置文件中对应的属性进行绑定的。

通过一张图标来理解一下这一繁复的流程：

![img](E:\markdown笔记\图片\how-spring-boot-autoconfigure-works.png) 图片来自于王福强老师的博客：<https://afoo.me/posts/2015-07-09-how-spring-boot-works.html> 

### 总结

综上是对自动配置原理的讲解。当然，在浏览源码的时候一定要记得不要太过拘泥与代码的实现，而是应该抓住重点脉络。

XxxxProperties类的含义是：封装配置文件中相关属性；

XxxxAutoConfiguration类的含义是：自动配置类，目的是给容器中添加组件。

而其他的主方法启动，则是为了加载这些五花八门的XxxxAutoConfiguration类。



## 配置文件加载顺序

> 1、开发者工具 `Devtools` 全局配置参数；
> 2、单元测试上的 `@TestPropertySource` 注解指定的参数；
> 3、单元测试上的 `@SpringBootTest` 注解指定的参数；
> 4、命令行指定的参数，如 `java -jar springboot.jar --name="Java技术栈"`；
> 5、命令行中的 `SPRING_APPLICATION_JSON` 指定参数, 如 `java -Dspring.application.json='{"name":"Java技术栈"}' -jar springboot.jar`
> 6、`ServletConfig` 初始化参数；
> 7、`ServletContext` 初始化参数；
> 8、JNDI参数（如 `java:comp/env/spring.application.json`）；
> 9、Java系统参数（来源：`System.getProperties()`）；
> 10、操作系统环境变量参数；
> 11、`RandomValuePropertySource` 随机数，仅匹配：`ramdom.*`；
> 12、JAR包外面的配置文件参数（`application-{profile}.properties（YAML）`）
> 13、JAR包里面的配置文件参数（`application-{profile}.properties（YAML）`）
> 14、JAR包外面的配置文件参数（`application.properties（YAML）`）
> 15、JAR包里面的配置文件参数（`application.properties（YAML）`）
> 16、`@Configuration`配置文件上 `@PropertySource` 注解加载的参数；
> 17、默认参数（通过 `SpringApplication.setDefaultProperties` 指定）；
>  
>

**数字小的优先级越高，即数字小的会覆盖数字大的参数值**

#### 一、存放目录

Application属性文件，按优先级排序，位置高的将覆盖位置

1. 当前项目目录下的一个/config子目录

2. 当前项目目录

3. 项目的resources即一个classpath下的/config包

4. 项目的resources即classpath根路径（root）
   

   

    如图：

   

   ![img](https:////upload-images.jianshu.io/upload_images/13184578-31bb8d4c59d678b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/478/format/webp)

   目录

#### 二、读取顺序

如果在不同的目录中存在多个配置文件，它的读取顺序是：
 1、config/application.properties（项目根目录中config目录下）
 2、config/application.yml
 3、application.properties（项目根目录下）
 4、application.yml
 5、resources/config/application.properties（项目resources目录中config目录下）
 6、resources/config/application.yml
 7、resources/application.properties（项目的resources目录下）
 8、resources/application.yml

顺序可以通过实验验证：

1~8 个位置 分别定义不同的 server 端口号 9001~9008
 即可验证结果顺序
 注：
 1、如果同一个目录下，有application.yml也有application.properties，默认先读取application.properties。
 2、如果同一个配置属性，在多个配置文件都配置了，默认使用第1个读取到的，后面读取的不覆盖前面读取到的。
 3、创建SpringBoot项目时，一般的配置文件放置在项目的resources目录下，因为配置文件的修改，通过热部署不用重新启动项目，而热部署的作用范围是classpath下

#### 三、配置文件的生效顺序会对值进行覆盖

1. @TestPropertySource 注解
2. 命令行参数
3. Java系统属性（System.getProperties()）
4. 操作系统环境变量
5. 只有在random.*里包含的属性会产生一个RandomValuePropertySource
6. 在打包的jar外的应用程序配置文件（application.properties，包含YAML和profile变量）
7. 在打包的jar内的应用程序配置文件（application.properties，包含YAML和profile变量）
8. 在@Configuration类上的@PropertySource注解
9. 默认属性（使用SpringApplication.setDefaultProperties指定）

#### 四、配置随机值

roncoo.secret={random.value} roncoo.number={random.int}
 roncoo.bignumber={random.long} roncoo.number.less.than.ten={random.int(10)}
 roncoo.number.in.range=${random.int[1024,65536]}

读取使用注解：@Value(value = "${roncoo.secret}")

注：出现黄点提示，是要提示配置元数据，可以不配置

#### 五、属性占位符

当application.properties里的值被使用时，它们会被存在的Environment过滤，所以你能够引用先前定义的值（比如，系统属性）
 [roncoo.name](http://roncoo.name) = [www.roncoo.com](http://www.roncoo.com)
 roncoo.desc = ${roncoo.name} is a domain name

#### 六、其他配置的介绍

- 端口配置
   server.port=8090
- 时间格式化
   spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
- 时区设置
   spring.jackson.time-zone=Asia/Chongqing



## SpringBoot 启动原理

### 一、引言

------

SpringBoot的一大优势就是Starter，由于SpringBoot有很多开箱即用的Starter依赖，使得我们开发变得简单，我们不需要过多的关注框架的配置。

在日常开发中，我们也会自定义一些Starter，特别是现在微服务框架，我们一个项目分成了多个单体项目，而这些单体项目中会引用公司的一些组件，这个时候我们定义Starter，可以使这些单体项目快速搭起，我们只需要关注业务开发。

在此之前我们再深入的了解下SpringBoot启动原理。而后再将如何自定义starter。

### 二、 启动原理

------

要想了解启动原理，我们可以Debug模式跟着代码一步步探究，我们从入口方法开始：

```java
public static ConfigurableApplicationContext run(Class<?>[] primarySources,
      String[] args) {
    return new SpringApplication(primarySources).run(args);
}
```

这里是创建一个SpringApplication对象，并调用了run方法

### 2.1 创建SpringApplication对象

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    //保存主配置类 
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    //确定web应用类型
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    //从类路径下找到META-INF/spring.factories配置的所有ApplicationContextInitializer；然后保存起来
    setInitializers((Collection) getSpringFactoriesInstances(
                  ApplicationContextInitializer.class));
    //从类路径下找到ETA-INF/spring.factories配置的所有ApplicationListener
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    //从多个配置类中找到有main方法的主配置类
    this.mainApplicationClass = deduceMainApplicationClass();
}
```

从这个方法中可以看出，这个

**第一步**：保存主配置类。

**第二步**：确定web应用类型。

**第三步**：setInitializers方法，这个方法走我们看带入的参数是getSpringFactoriesInstances(ApplicationContextInitializer.class)，我们再往下查看getSpringFactoriesInstances



![img](E:\markdown笔记\图片\13449584-49aed14ce949deca.webp)

再进入这个方法：



![img](E:\markdown笔记\图片\13449584-4c76f5f4ae36ee55.webp)

这里就是从类路径下找到META-INF/spring.factories配置的所有ApplicationContextInitializer，然后再保存起来，放开断点，我们可以看到这个时候获取到的



![img](E:\markdown笔记\图片\13449584-3259b21901252fed.webp)

**第四步**：从类路径下找到ETA-INF/spring.factories配置的所有ApplicationListener，原理也基本类似,进入断点



![img](E:\markdown笔记\图片\13449584-762e63093d8803ce.webp)

**第五步**：从多个配置类中找到有main方法的主配置类。这个执行完之后，SpringApplication就创建完成

## 2.2 run方法

先贴出代码

```java
public ConfigurableApplicationContext run(String... args) {
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    ConfigurableApplicationContext context = null;
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    configureHeadlessProperty();
    //从类路径下META-INF/spring.factories获取SpringApplicationRunListeners
    SpringApplicationRunListeners listeners = getRunListeners(args);
    //回调所有的获取SpringApplicationRunListener.starting()方法
    listeners.starting();
    try {
        //封装命令行参数
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(
                    args);
        //准备环境 
        ConfigurableEnvironment environment = prepareEnvironment(listeners,
                    applicationArguments);
        //创建环境完成后回调SpringApplicationRunListener.environmentPrepared()；表示环境准备完成
        configureIgnoreBeanInfo(environment);
        //打印Banner图
        Banner printedBanner = printBanner(environment);
        //创建ApplicationContext,决定创建web的ioc还是普通的ioc  
        context = createApplicationContext();
        //异常分析报告
        exceptionReporters = getSpringFactoriesInstances(
                    SpringBootExceptionReporter.class,
                    new Class[] { ConfigurableApplicationContext.class }, context);
        //准备上下文环境，将environment保存到ioc中
        //applyInitializers()：回调之前保存的所有的ApplicationContextInitializer的initialize方法 
        //listeners.contextPrepared(context) 
        //prepareContext运行完成以后回调所有的SpringApplicationRunListener的contextLoaded（）
        prepareContext(context, environment, listeners, applicationArguments,
                    printedBanner);
        //刷新容器,ioc容器初始化（如果是web应用还会创建嵌入式的Tomcat）
        //扫描，创建，加载所有组件的地方,（配置类，组件，自动配置）
        refreshContext(context);
        afterRefresh(context, applicationArguments);
        stopWatch.stop();
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass)
                           .logStarted(getApplicationLog(), stopWatch);
        }
        //所有的SpringApplicationRunListener回调started方法
        listeners.started(context);
        //从ioc容器中获取所有的ApplicationRunner和CommandLineRunner进行回调，
        //ApplicationRunner先回调，CommandLineRunner再回调
        callRunners(context, applicationArguments);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, listeners);
        throw new IllegalStateException(ex);
    }
    try {
        //所有的SpringApplicationRunListener回调running方法
        listeners.running(context);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, null);
        throw new IllegalStateException(ex);
    }
    //整个SpringBoot应用启动完成以后返回启动的ioc容器
    return context;
}
```

前面的代码不用分析，主要是准备对象，我们从 SpringApplicationRunListeners listeners = getRunListeners(args)开始分析，

**第一步**：是从类路径下META-INF/spring.factories获取SpringApplicationRunListeners，

这个方法跟前面分析的两个获取配置方法类似。

**第二步**：回调所有的获取SpringApplicationRunListener.starting()方法。

**第三步**： 封装命令行参数。

**第四步**：准备环境，调用prepareEnvironment方法。

**第五步**：打印Banner图（就是启动时的标识图）。

**第六步**：创建ApplicationContext,决定创建web的ioc还是普通的ioc。

**第七步**：异常分析报告。

**第八步**：准备上下文环境，将environment保存到ioc中，这个方法需要仔细分析下，我们再进入这个方法



![img](E:\markdown笔记\图片\13449584-78c395fce699dfeb.webp)

这里面有一个applyInitializers方法,这里是回调之前保存的所有的ApplicationContextInitializer的initialize方法



![img](E:\markdown笔记\图片\13449584-63c1845323f9df10.webp)

还有一个listeners.contextPrepared(context)，这里是回调所有的SpringApplicationRunListener的contextPrepared()，

最后listeners.contextLoaded(context) 是prepareContext运行完成以后回调所有的SpringApplicationRunListener的contextLoaded（）。

**第九步**：刷新容器,ioc容器初始化（如果是web应用还会创建嵌入式的Tomcat）,这个就是扫描，创建，加载所有组件的地方,（配置类，组件，自动配置）。

**第十步**：所有的SpringApplicationRunListener回调started方法。

**第十一步**：从ioc容器中获取所有的ApplicationRunner和CommandLineRunner进行回调，ApplicationRunner先回调，CommandLineRunner再回调。

**第十二步**：所有的SpringApplicationRunListener回调running方法。

**第十三步**：整个SpringBoot应用启动完成以后返回启动的ioc容器。

这就是run的全部过程，想要更详细的了解还需自己去看源码。



## SpringBoot中的Starters

Spring Boot Starter是在SpringBoot组件中被提出来的一种概念，stackoverflow上面已经有人概括了这个starter是什么东西，想看完整的回答戳[这里](https://stackoverflow.com/a/28273660)

> Starter POMs are a set of convenient dependency descriptors that you can include in your application. You get a one-stop-shop for all the Spring and related technology that you need, without having to hunt through sample code and copy paste loads of dependency descriptors. For example, if you want to get started using Spring and JPA for database access, just include the spring-boot-starter-data-jpa dependency in your project, and you are good to go.

大概意思就是说starter是一种对依赖的 synthesize（合成），这是什么意思呢？我可以举个例子来说明。

### 传统的做法

在没有starter之前，假如我想要在Spring中使用jpa，那我可能需要做以下操作：

1. 在Maven中引入使用的数据库的依赖（即JDBC的jar）
2. 引入jpa的依赖
3. 在xxx.xml中配置一些属性信息
4. 反复的调试直到可以正常运行

需要注意的是，这里操作在我们**每次新建一个需要用到jpa的项目的时候都需要重复的做一次**。也许你在第一次自己建立项目的时候是在Google上自己搜索了一番，花了半天时间解决掉了各种奇怪的问题之后，jpa终于能正常运行了。有些有经验的人会在OneNote上面把这次建立项目的过程给记录下来，包括操作的步骤以及需要用到的配置文件的内容，在下一次再创建jpa项目的时候，就不需要再次去Google了，只需要照着笔记来，之后再把所有的配置文件copy&paste就可以了。

像上面这样的操作也不算不行，事实上我们在没有starter之前都是这么干的，但是这样做有几个问题：

1. 如果过程比较繁琐，这样一步步操作会增加出错的可能性
2. 不停地copy&paste不符合[Don’t repeat yourself](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)精神
3. 在第一次配置的时候（尤其如果开发者比较小白），需要花费掉大量的时间

### 使用Spring Boot Starter提升效率

starter的主要目的就是为了解决上面的这些问题。

#### starter的理念

starter会把所有用到的依赖都给包含进来，避免了开发者自己去引入依赖所带来的麻烦。需要注意的是不同的starter是为了解决不同的依赖，所以它们内部的实现可能会有很大的差异，例如jpa的starter和Redis的starter可能实现就不一样，**这是因为starter的本质在于synthesize，这是一层在逻辑层面的抽象**，也许这种理念有点类似于Docker，因为它们都是在做一个“包装”的操作，如果你知道Docker是为了解决什么问题的，也许你可以用Docker和starter做一个类比。

#### starter的实现

虽然不同的starter实现起来各有差异，但是他们基本上都会使用到两个相同的内容：**ConfigurationProperties**和**AutoConfiguration**。因为Spring Boot坚信“约定大于配置”这一理念，所以我们使用ConfigurationProperties来保存我们的配置，并且这些配置都可以有一个默认值，即在我们没有主动覆写原始配置的情况下，默认值就会生效，这在很多情况下是非常有用的。

除此之外，starter的ConfigurationProperties还使得所有的配置属性被聚集到一个文件中（一般在resources目录下的application.properties），这样我们就告别了Spring项目中XML地狱。

starter的整体逻辑：

![img](E:\markdown笔记\图片\20190625155410842.png)

上面的starter依赖的jar和我们自己手动配置的时候依赖的jar并没有什么不同，所以我们可以认为starter其实是把这一些繁琐的配置操作交给了自己，而把简单交给了用户。除了帮助用户去除了繁琐的构建操作，在“约定大于配置”的理念下，`ConfigurationProperties`还帮助用户减少了无谓的配置操作。并且因为 `application.properties` 文件的存在，即使需要自定义配置，所有的配置也只需要在一个文件中进行，使用起来非常方便。

了解了starter其实就是帮助用户简化了配置的操作之后，要理解starter和被配置了starter的组件之间并不是竞争关系，而是**辅助关系**，即我们可以给一个组件创建一个starter来让最终用户在使用这个组件的时候更加的简单方便。基于这种理念，我们可以给任意一个现有的组件创建一个starter来让别人在使用这个组件的时候更加的简单方便，事实上Spring Boot团队已经帮助现有大部分的流行的组件创建好了它们的starter，你可以在[这里](https://github.com/spring-projects/spring-boot/tree/v1.5.7.RELEASE/spring-boot-starters)查看这些starter的列表。

### 创建自己的Spring Boot Starter

如果你想要自己创建一个starter，那么基本上包含以下几步

1. 创建一个starter项目，关于项目的命名你可以参考[这里](https://docs.spring.io/spring-boot/docs/2.0.0.M5/reference/htmlsingle/#boot-features-custom-starter-naming)
2. 创建一个ConfigurationProperties用于保存你的配置信息（如果你的项目不使用配置信息则可以跳过这一步，不过这种情况非常少见）
3. 创建一个AutoConfiguration，引用定义好的配置信息；在AutoConfiguration中实现所有starter应该完成的操作，并且把这个类加入spring.factories配置文件中进行声明
4. 打包项目，之后在一个SpringBoot项目中引入该项目依赖，然后就可以使用该starter了

我们来看一个例子（例子的完整代码位于<https://github.com/RitterHou/learn-spring-boot-starter>）

首先新建一个Maven项目，设置 `pom.xml` 文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <artifactId>http-starter</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <!-- 自定义starter都应该继承自该依赖 -->
    <!-- 如果自定义starter本身需要继承其它的依赖，可以参考 https://stackoverflow.com/a/21318359 解决 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starters</artifactId>
        <version>1.5.2.RELEASE</version>
    </parent>
    <dependencies>
        <!-- 自定义starter依赖此jar包 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <!-- lombok用于自动生成get、set方法 -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.16.10</version>
        </dependency>
    </dependencies>
</project>
```


创建proterties类来保存配置信息：

```java
@ConfigurationProperties(prefix = "http") // 自动获取配置文件中前缀为http的属性，把值传入对象参数
@Setter
@Getter
public class HttpProperties {
    // 如果配置文件中配置了http.url属性，则该默认属性会被覆盖
    private String url = "http://www.baidu.com/";
}
```


上面这个类就是定义了一个属性，其默认值是 `http://www.baidu.com/`，我们可以通过在 `application.properties` 中添加配置 `http.url=https://www.zhihu.com` 来覆盖参数的值。

创建业务类：

```java
@Setter
@Getter
public class HttpClient {
    private String url;
    // 根据url获取网页数据
    public String getHtml() {
        try {
            URL url = new URL(this.url);
            URLConnection urlConnection = url.openConnection();
            BufferedReader br = new BufferedReader(new 														InputStreamReader(urlConnection.getInputStream(), "utf-8"));
            String line = null;
            StringBuilder sb = new StringBuilder();
            while ((line = br.readLine()) != null) {
                sb.append(line).append("\n");
            }
            return sb.toString();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return "error";
    }
}
```


这个业务类的操作非常简单，只包含了一个 `url` 属性和一个 `getHtml` 方法，用于获取一个网页的HTML数据，读者看看就懂了。

创建AutoConfiguration

```java
@Configuration
@EnableConfigurationProperties(HttpProperties.class)
public class HttpAutoConfiguration {
    @Resource
    private HttpProperties properties; // 使用配置
    // 在Spring上下文中创建一个对象
    @Bean
    @ConditionalOnMissingBean
    public HttpClient init() {
        HttpClient client = new HttpClient();
        String url = properties.getUrl();
        client.setUrl(url);
        return client;
    }
}
```


在上面的AutoConfiguration中我们实现了自己要求：在Spring的上下文中创建了一个HttpClient类的bean，并且我们把properties中的一个参数赋给了该bean。

关于`@ConditionalOnMissingBean` 这个注解，它的意思是在该bean不存在的情况下此方法才会执行，这个相当于开关的角色，更多关于开关系列的注解可以参考[这里](https://docs.spring.io/spring-boot/docs/2.0.0.M5/reference/htmlsingle/#boot-features-condition-annotations)。

最后，我们在 `resources` 文件夹下新建目录 `META-INF`，在目录中新建 `spring.factories` 文件，并且在 `spring.factories` 中配置AutoConfiguration：

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.nosuchfield.httpstarter.HttpAutoConfiguration
```

到此，我们的starter已经创建完毕了，使用Maven打包该项目。之后创建一个SpringBoot项目，在项目中添加我们之前打包的starter作为依赖，然后使用SringBoot来运行我们的starter，代码如下：

```java
@Component
public class RunIt {
    @Resource
    private HttpClient httpClient;
    public void hello() {
        System.out.println(httpClient.getHtml());
    }
}
```


 正常情况下此方法的执行会打印出url `http://www.baidu.com/` 的HTML内容，之后我们在application.properties中加入配置：

```xml
http.url=https://www.zhihu.com/
```

再次运行程序，此时打印的结果应该是知乎首页的HTML了，证明properties中的数据确实被覆盖了。