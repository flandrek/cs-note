#### 1. Java-8 新特性

**一、Lambda表达式**

Lambda表达式可以说是Java 8最大的卖点，她将函数式编程引入了Java。Lambda允许把函数作为一个方法的参数，或者把代码看成数据。

一个Lambda表达式可以由用逗号分隔的参数列表、–>符号与函数体三部分表示。例如：

Arrays.asList( "p", "k", "u","f", "o", "r","k").forEach( e -> System.out.println( e ) );

 1 Arrays.asList( "p", "k", "u","f", "o", "r","k").forEach( e -> System.out.println( e ) ); 

为了使现有函数更好的支持Lambda表达式，Java 8引入了函数式接口的概念。函数式接口就是只有一个方法的普通接口。java.lang.Runnable与java.util.concurrent.Callable是函数式接口最典型的例子。为此，Java 8增加了一种特殊的注解@FunctionalInterface：

```java
1 @FunctionalInterface
2 public interface Functional {
3     void method();
4 }
```

**二、接口的默认方法与静态方法**

我们可以在接口中定义默认方法，使用default关键字，并提供默认的实现。所有实现这个接口的类都会接受默认方法的实现，除非子类提供的自己的实现。例如：

```java
1 public interface DefaultFunctionInterface {
2     default String defaultFunction() {
3         return "default function";
4     }
5 }
```

我们还可以在接口中定义静态方法，使用static关键字，也可以提供实现。例如：

```java
1 public interface StaticFunctionInterface {
2     static String staticFunction() {
3         return "static function";
4     }
5 }
```

接口的默认方法和静态方法的引入，其实可以认为引入了C＋＋中抽象类的理念，以后我们再也不用在每个实现类中都写重复的代码了。

**三、方法引用**

通常与Lambda表达式联合使用，可以直接引用已有Java类或对象的方法。一般有四种不同的方法引用：

1. 构造器引用。语法是Class::new，或者更一般的Class< T >::new，要求构造器方法是没有参数；
2. 静态方法引用。语法是Class::static_method，要求接受一个Class类型的参数；
3. 特定类的任意对象方法引用。它的语法是Class::method。要求方法是没有参数的；
4. 特定对象的方法引用，它的语法是instance::method。要求方法接受一个参数，与3不同的地方在于，3是在列表元素上分别调用方法，而4是在某个对象上调用方法，将列表元素作为参数传入；

**四、重复注解**

在Java 5中使用注解有一个限制，即相同的注解在同一位置只能声明一次。Java 8引入重复注解，这样相同的注解在同一地方也可以声明多次。重复注解机制本身需要用@Repeatable注解。Java 8在编译器层做了优化，相同注解会以集合的方式保存，因此底层的原理并没有变化。

**五、扩展注解的支持**

Java 8扩展了注解的上下文，几乎可以为任何东西添加注解，包括局部变量、泛型类、父类与接口的实现，连方法的异常也能添加注解。

**六、Optional**

Java 8引入Optional类来防止空指针异常，Optional类最先是由Google的Guava项目引入的。Optional类实际上是个容器：它可以保存类型T的值，或者保存null。使用Optional类我们就不用显式进行空指针检查了。

**七、Stream**

Stream API是把真正的函数式编程风格引入到Java中。其实简单来说可以把Stream理解为MapReduce，当然Google的MapReduce的灵感也是来自函数式编程。她其实是一连串支持连续、并行聚集操作的元素。从语法上看，也很像linux的管道、或者链式编程，代码写起来简洁明了，非常酷帅！

**八、Date/Time API (JSR 310)**

Java 8新的Date-Time API (JSR 310)受Joda-Time的影响，提供了新的java.time包，可以用来替代 java.util.Date和java.util.Calendar。一般会用到Clock、LocaleDate、LocalTime、LocaleDateTime、ZonedDateTime、Duration这些类，对于时间日期的改进还是非常不错的。

**九、JavaScript引擎Nashorn**

Nashorn允许在JVM上开发运行JavaScript应用，允许Java与JavaScript相互调用。

**十、Base64**

在Java 8中，Base64编码成为了Java类库的标准。Base64类同时还提供了对URL、MIME友好的编码器与解码器。

除了这十大新特性之外，还有另外的一些新特性：

- **更好的类型推测机制**：Java 8在类型推测方面有了很大的提高，这就使代码更整洁，不需要太多的强制类型转换了。
- **编译器优化**：Java 8将方法的参数名加入了字节码中，这样在运行时通过反射就能获取到参数名，只需要在编译时使用-parameters参数。
- **并行（parallel）数组**：支持对数组进行并行处理，主要是parallelSort()方法，它可以在多核机器上极大提高数组排序的速度。
- **并发（Concurrency）**：在新增Stream机制与Lambda的基础之上，加入了一些新方法来支持聚集操作。
- **Nashorn引擎jjs**：基于Nashorn引擎的命令行工具。它接受一些JavaScript源代码为参数，并且执行这些源代码。
- **类依赖分析器jdeps**：可以显示Java类的包级别或类级别的依赖。
- **JVM的PermGen空间被移除**：取代它的是Metaspace（JEP 122）。

Java 8是一次变化巨大的更新，耗费了工程师大量的时间，还借鉴了很多其它语言和类库。我们无法在这里一一详细列举，以后有机会一定给大家详细解读一下。



#### 2. JDK、JRE、JVM区别与联系

**关键字：JDK，JRE，JVM**
摘要：JDK是 Java 语言的软件开发工具包(SDK)。在JDK的安装目录下有一个jre目录，里面有两个文件夹bin和lib，在这里可以认为bin里的就是jvm，lib中则是jvm工作所需要的类库，而jvm和 lib合起来就称为jre。

**一、JDK**
JDK(Java Development Kit) 是整个JAVA的核心，包括了Java运行环境（Java Runtime Envirnment），一堆Java工具（javac/java/jdb等）和Java基础的类库（即Java API 包括rt.jar）。
JDK是java开发工具包，基本上每个学java的人都会先在机器 上装一个JDK，那他都包含哪几部分呢？在目录下面有 六个文件夹、一个src类库源码压缩包、和其他几个声明文件。其中，真正在运行java时起作用的 是以下四个文件夹：bin、include、lib、 jre。有这样一个关系，JDK包含JRE，而JRE包 含JVM。

      bin:最主要的是编译器(javac.exe)
      include:java和JVM交互用的头文件
      lib：类库
      jre:java运行环境
（注意：这里的bin、lib文件夹和jre里的bin、lib是 不同的）

总的来说JDK是用于java程序的开发,而jre则是只能运行class而没有编译的功能。

**二、JRE**

JRE（Java Runtime Environment，Java运行环境），包含JVM标准实现及Java核心类库。JRE是Java运行环境，并不是一个开发环境，所以没有包含任何开发工具（如编译器和调试器）
JRE是指java运行环境。光有JVM还不能成class的 执行，因为在解释class的时候JVM需要调用解释所需要的类库lib。 （jre里有运行.class的java.exe）
JRE （ Java Runtime Environment ），是运行 Java 程序必不可少的（除非用其他一些编译环境编译成.exe可执行文件……），JRE的 地位就象一台PC机一样，我们写好的Win64应用程序需要操作系统帮 我们运行，同样的，我们编写的Java程序也必须要JRE才能运行。

**三、JVM**

JVM（Java Virtual Machine），即java虚拟机, java运行时的环境，JVM是一种用于计算设备的规范，它是一个虚构出来的计算机，是通过在实际的计算机上仿真模拟各种计算机功能来实现的。针对java用户，也就是拥有可运行的.class文件包（jar或者war）的用户。里面主要包含了jvm和java运行时基本类库（rt.jar）。rt.jar可以简单粗暴地理解为：它就是java源码编译成的jar包。Java虚拟机在执行字节码时，把字节码解释成具体平台上的机器指令执行。这就是Java的能够“一次编译，到处运行”的原因。

**四、JDK、JRE、JVM三者的联系与区别**

1.三者联系：
JVM不能单独搞定class的执行，解释class的时候JVM需要调用解释所需要的类库lib。在JDK下面的的jre目录里面有两个文件夹bin和lib,在这里可以认为bin里的就是jvm，lib中则是jvm工作所需要的类库，而jvm和 lib和起来就称为jre。JVM+Lib=JRE。总体来说就是，我们利用JDK（调用JAVA API）开发了属于我们自己的JAVA程序后，通过JDK中的编译程序（javac）将我们的文本java文件编译成JAVA字节码，在JRE上运行这些JAVA字节码，JVM解析这些字节码，映射到CPU指令集或OS的系统调用。

2.三者区别：
	a.JDK和JRE区别：在bin文件夹下会发现，JDK有javac.exe而JRE里面没有，javac指令是用来将java文件编译成class文件的，这是开发者需要的，而用户（只需要运行的人）是不需要的。JDK还有jar.exe, javadoc.exe等等用于开发的可执行指令文件。这也证实了一个是开发环境，一个是运行环境。

​	b.JRE和JVM区别：JVM并不代表就可以执行class了，JVM执行.class还需要JRE下的lib类库的支持，尤其是rt.jar。



#### 3. Java中四种引用的区别

ava将引用分为强引用、软引用、弱引用和虚引用四种，这四种引用强度依次逐渐减弱。

下面分别从四种引用的基本概念、具体实现、生命周期和应用场景四个方面进行对比分析。

**1.强引用（Strong Reference）：**

```html
基本概念：在程序代码中普遍存在的，类似于“Object obj = new Object()”形式的引用。
具体实现：创建一个对象并将其赋值给一个引用变量。
生命周期：垃圾收集器永远不会回收掉强引用所关联着的对象，除非将该对象的所有引用全部置为null。
应用场景：用的太多，以至于我说不出来（撒手状）。
```

**2.软引用（Soft Reference）：**

```html
基本概念：描述一些还有用，但并非必需的对象。
具体实现：JDK提供了SoftReference类来实现。
生命周期：在系统将要发生内存溢出异常之前，将会把软引用所关联着的对象列进回收范围内并进行第二次回收，若回收后仍没有足够内存则抛出内存溢出异常。
应用场景：用于实现内存敏感的高速缓存，如网页缓存和图片缓存等。
注意：使用软引用能防止内存泄漏，增强程序的健壮性。
```

**3.弱引用（Weak Reference）：**

```html
基本概念：也描述非必需对象，强度比软引用更弱。
具体实现：JDK提供了WeakReference类来实现。
生命周期：无论系统内存是否足够，垃圾收集器工作时都会回收只被弱引用关联着的对象。
应用场景：多将对象的弱引用作为HashMap的key值，以实现在不需要该对象时，只需要在程序中将其强引用置为null，而无需手动将其从HashMap中移除（由垃圾收集器实现，WeakHashMap即是基于该原理）。
```

**4.虚引用（Phantom Reference）：**

```html
基本概念：也称幽灵引用或幻影引用，是最弱的一种引用关系。
具体实现：JDK提供了PhantomReference类来实现。
生命周期：虚引用并不会影响其所关联对象的生存时间，也无法通过虚引用来取得一个对象实例。
```

**问：Java扩展为四种引用的目的是什么？**

- 有利于Java虚拟机进行垃圾回收；
- 可以让程序员通过代码的方式来决定某些对象的生命周期。



#### 4.注解

**1.什么是注解？**

对于很多初次接触的开发者来说应该都有这个疑问？***Annontation***是Java5开始引入的新特征，中文名称叫**注解**。它提供了一种安全的类似注释的机制，用来将任何的信息或元数据（metadata）与程序元素（类、方法、成员变量等）进行关联。为程序的元素（类、方法、成员变量）加上更直观更明了的说明，这些说明信息是与程序的业务逻辑无关，并且供指定的工具或框架使用。Annontation像一种修饰符一样，应用于包、类型、构造方法、方法、成员变量、参数及本地变量的声明语句中。
　　Java注解是附加在代码中的一些元信息，用于一些工具在编译、运行时进行解析和使用，起到说明、配置的功能。注解不会也不能影响代码的实际逻辑，仅仅起到辅助性的作用。包含在 java.lang.annotation 包中。

 **2.注解的用处**

- 生成文档。这是最常见的，也是java 最早提供的注解。常用的有@param @return 等
- 跟踪代码依赖性，实现替代配置文件功能。比如Dagger 2 依赖注入，未来java 开发，将大量注解配置，具有很大用处;
- 在编译时进行格式检查。如@override 放在方法前，如果你这个方法并不是覆盖了超类方法，则编译时就能检查出。

**3.注解原理**

注解本质是一个继承了Annotation 的特殊接口，其具体实现类是Java 运行时生成的动态代理类。而我们通过反射获取注解时，返回的是Java 运行时生成的动态代理对象$Proxy1。通过代理对象调用自定义注解（接口）的方法，会最终调用AnnotationInvocationHandler 的invoke 方法。该方法会从memberValues 这个Map 中索引出对应的值。而memberValues 的来源是Java 常量池。