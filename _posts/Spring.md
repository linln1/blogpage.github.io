---
layout: post
---

摘要
<!--more-->
<!-- CreateTime:2020/6/29 18:26:39 -->


<div id="toc"></div>
#Java笔记
##Spring
###IoC容器
IoC全称Inversion of Control，直译为控制反转 
    核心问题是：

    谁负责创建组件？
    谁负责根据依赖关系组装组件？
    销毁时，如何按依赖顺序正确销毁？

    在IoC模式下，控制权发生了反转，即从应用程序转移到了IoC容器，所有组件不再由应用程序自己创建和配置，而是由IoC容器负责，这样，应用程序只需要直接使用已经创建好并且配置好的组件

    需要某种“注入”机制，例如，BookService自己并不会创建DataSource，而是等待外部通过setDataSource()方法来注入一个DataSource

    IoC又称为依赖注入（DI：Dependency Injection），它解决了一个最主要的问题：将组件的创建+配置与组件的使用相分离，并且，由IoC容器负责管理组件的生命周期

    IoC容器要负责实例化所有的组件，因此，有必要告诉容器如何创建组件，以及各组件的依赖关系。一种最简单的配置是通过XML文件来实现，例如：

    <beans>
        <bean id="dataSource" class="HikariDataSource" />
        <bean id="bookService" class="BookService">
            <property name="dataSource" ref="dataSource" />
        </bean>
        <bean id="userService" class="UserService">
            <property name="dataSource" ref="dataSource" />
        </bean>
    </beans>

    上述XML配置文件指示IoC容器创建3个JavaBean组件，并把id为dataSource的组件通过属性dataSource（即调用setDataSource()方法）注入到另外两个组件中。

    在Spring的IoC容器中，我们把所有组件统称为JavaBean，即配置一个组件就是配置一个Bean
####无侵入容器
    在设计上，Spring的IoC容器是一个高度可扩展的无侵入容器。所谓无侵入，是指应用程序的组件无需实现Spring的特定接口，或者说，组件根本不知道自己在Spring的容器中运行。这种无侵入的设计有以下好处：

    应用程序组件既可以在Spring的IoC容器中运行，也可以自己编写代码自行组装配置；
    测试的时候并不依赖Spring容器，可单独进行测试，大大提高了开发效率

####装配Bean
    Application.xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
            https://www.springframework.org/schema/beans/spring-beans.xsd">

        <bean id="userService" class="com.itranswarp.learnjava.service.UserService">
            <property name="mailService" ref="mailService" />
        </bean>

        <bean id="mailService" class="com.itranswarp.learnjava.service.MailService" />
    </beans>
    public class Main {
        public static void main(String[] args) {
            ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
            UserService userService = context.getBean(UserService.class);
            User user = userService.login("bob@example.com", "password");
            System.out.println(user.getName());
        }
    }
####BeanFactory
    Spring还提供另一种IoC容器叫BeanFactory，使用方式和ApplicationContext类似：

    BeanFactory factory = new XmlBeanFactory(new ClassPathResource("application.xml"));
    MailService mailService = factory.getBean(MailService.class);
    BeanFactory和ApplicationContext的区别在于，BeanFactory的实现是按需创建，即第一次获取Bean时才创建这个Bean，而ApplicationContext会一次性创建所有的Bean。实际上，ApplicationContext接口是从BeanFactory接口继承而来的，并且，ApplicationContext提供了一些额外的功能，包括国际化支持、事件和通知机制等。通常情况下，我们总是使用ApplicationContext，很少会考虑使用BeanFactory

####简化Application.xml配置______Annotation



###AOP面向切面编程
   Proxy模式
   将某个功能，比如权限检查,安全检查,日志,事务等功能
   Proxy
   public class SecurityCheckBookService implements BookService{
       public final BookService target;

       public SecurityCheckBookService(BookService target){
           this.target = target;
       }

       public void createBook(Book book){
           securityCheck();
           target.createBook(book);
       }
   }

    AOP的织入，有3种方式：

    1)编译期：在编译时，由编译器把切面调用编译进字节码，这种方式需要定义新的关键字并扩展编译器，AspectJ就扩展了Java编译器，使用关键字aspect来实现织入；
    2)类加载器：在目标类被装载到JVM时，通过一个特殊的类加载器，对目标类的字节码重新“增强”；
    3)运行期：目标对象和切面都是普通Java类，通过JVM的动态代理功能或者第三方库实现运行期动态织入


