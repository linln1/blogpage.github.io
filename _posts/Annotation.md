---
layout: post
---

摘要
<!--more-->
<!-- CreateTime:2020/6/27 11:23:54 -->


<div id="toc"></div>

#Java笔记
##Annotation
    注解是放在Java源码的类、方法、字段、参数前的一种特殊“注释”：
    注解则可以被编译器打包进入class文件，因此，注解是一种用作标注的“元数据

    Java的注解可以分为三类：
    1)由编译器使用的注解:@Override/@SuppressWarnings
        这类注解不会被编译进入.class文件，它们在编译后就被编译器扔掉了。
    2)工具处理.class文件使用的注解,加载class的时候，对class做动态修改，实现一些特殊功能，会被编译进入.class文件，加载结束后并不会存在于内存中.
    3)在程序运行期能够读取的注解，它们在加载后一直存在于JVM中
    @PostConstruct(Java代码读取该注解实现的功能，JVM并不会识别该注解)
    定义一个注解时，还可以定义配置参数。配置参数可以包括：

        所有基本类型；
        String；
        枚举类型；
        基本类型、String、Class以及枚举的数组。
        因为配置参数必须是常量，所以，上述限制保证了注解在定义时就已经确定了每个参数的值。

    注解的配置参数可以有默认值，缺少某个配置参数时将使用默认值。

    此外，大部分注解会有一个名为value的配置参数，对此参数赋值，可以只写常量，相当于省略了value参数。

    如果只写注解，相当于全部使用默认值。

    Java语言使用@interface语法来定义注解（Annotation）
    public @interface Report {
        int type() default 0;
        String level() default "info";
        String value() default "";
    }

###元注解
    有一些注解可以修饰其他注解
    ####@Target
    可以定义Annotation能够被应用于源码的哪些位置：
        类或接口：ElementType.TYPE；
        字段：ElementType.FIELD；
        方法：ElementType.METHOD；
        构造方法：ElementType.CONSTRUCTOR；
        方法参数：ElementType.PARAMETER
    eg: Report 可以用在方法的注解上
    @Target(ElementType.Method)
    public @interface Report{
        int type() default 0;
        String level() default "info";
        String value() default "";
    }
    实际上@Target定义的value是ElementType[]数组，只有一个元素时，可以省略数组的写法。
    ####@Retention
    定义了Annotation的生命周期
        仅编译期：RetentionPolicy.SOURCE；
        仅class文件：RetentionPolicy.CLASS；
        运行期：RetentionPolicy.RUNTIME
    @Retention不存在，则该Annotation默认为CLASS,通常我们自定义的Annotation都是RUNTIME，所以，务必要加上@Retention(RetentionPolicy.RUNTIME)这个元注解
    @Rentention(RententionPolicy.RUNTIME)
    ####@Repeatable
    可以定义Annotation是否可重复。这个注解应用不是特别广泛。
    经过@Repeatable修饰后，在某个类型声明处，就可以添加多个@Report注解：
    @Repeatable(Reports.class)
    @Target(ElementType.TYPE)
    ####@Inherited
    定义子类是否可继承父类定义的Annotation。@Inherited仅针对@Target(ElementType.TYPE)类型的annotation有效，并且仅针对class的继承，对interface的继承无效：

###如何定义Annotation
    1)用@interface定义注释
    2)添加参数和默认值
    3)使用元注解配置注释

###如何处理注解
    根据@Retention的配置：
    SOURCE类型的注解在编译期就被丢掉了；
    CLASS类型的注解仅保存在class文件中，它们不会被加载进JVM；
    RUNTIME类型的注解会被加载进JVM，并且在运行期可以被程序读取。

    注解定义后也是一种class，所有的注解都继承自java.lang.annotation.Annotation，因此，读取注解，需要使用反射API

    判断某个注解是否存在于Class、Field、Method或Constructor：
        Class.isAnnotationPresent(Class)
        Field.isAnnotationPresent(Class)
        Method.isAnnotationPresent(Class)
        Constructor.isAnnotationPresent(Class)
    
