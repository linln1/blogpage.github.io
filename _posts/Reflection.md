---
layout: post
---

摘要
<!--more-->
<!-- CreateTime:2020/6/24 21:21:21 -->


<div id="toc"></div>

#Java笔记
##Reflection

###Class
    class 是 JVM 运行的时候动态加载的，JVM 第一次读到class类型就把它加入内存
    随后 JVM 创建一个 Class 类型的实例， 并且挂念起来

    这个Class类型如下
    public class Class{
        private Class() {}
    }

    JVM 加载 String 类的时候，他先读取String.class到内存
    Class cls = new Class(String);
    Class 实例 cls 中包含了该 class(String) 的所有完整信息
    name = "java.lang.String";
    package = "java.lang";
    super = "java.lang.Object";
    interface = CharSequence...
    field = value[], hash, ...
    method = indexOf(), ...

    通过Class 实例 cls 获取 class信息的方法叫 反射
    1) Class cls = String.class;
    2) String s = "Hello", Class cls = s.getClass();
    3) Class cls = Class.forName("java.lang.String");

    Class cls = String.class;
    String s = (String) cls.newInstance();

####动态加载
    JVM在执行程序的时候并不是一口气把所有用到的class全部加载到内存，而是第一次用class的时候才加载
    Commons Logging 总是优先使用 Log4j，JDK的logging

####访问字段
    import java.lang.reflect Field
    public class Main{
        public static void main(String[] args) throws Execption{
            Object p = new Person("Xiao ming");
            Class c = p.getClass();
            Field f = c.getDeclaredField("name");
            f.setAccessible(true);
            Object value = f.get(p);
            System.out.println(value);
        }
    }
    class Person{
        private String name;

        public Person(String name){
            this.name = name;
        }
    }

####调用方法
    Class stdClass = Student.class
    Method m = std.Class.getMethod("getScore", String.class);
    m.invoke(实例对象,String s);

    #####调用静态方法
    m.invoke(null, String s);//静态方法不用指定实例对象

    #####调用非public方法
    Method.setAccessible(true);

####调用构造函数
    用Class.class.newInstance()只可以调用无参public构造方法;
    constructor 获得的东西与父类无关
    import java.lang.reflect.Constructor;
    public class Main{
        Constructor cons1 = Integer.class.getConstructor(int.class);
        Integer n1 = (Integer) cons1.newInstance(1);
    }

####继承关系
    有了Class实例，我们还可以获取它的父类Class,getSuperClass()
    public class Main{
        public static void main(Stringp[] args) throws Exception{
            Class i = Integet.class;
            Class n = i.getSuperClass();
            System.out.println(n);
            Class o = n.getSuperClass();
            System.out.println(o);

        }
    }
    获取interface,查询一个类所实现到的多个接口
    s.getInterfaces();
    isAssignableFrom() 判断一个向上转型是否可以实现

####动态代理Dynamic Proxy
    所有的interface **不可实例化**
    interface类型的变量总是通过向上转型指向某个实例
    那么不编写实现类，直接在运行期创建某个interface的实例，用动态代理的机制：可以在运行期间动态创建某个interface的实例
    eg:
    静态
        public interface Hello{
            void morning(String name);
        }

        public class HelloWorld implements Hello{
            public void morning(String name){
                System.out.println("xxx" + name);
            }
        }

        Hello hello = new HelloWorld();
        hello.morning("me");
    动态
        import java.lang.reflect.InvocationHandler;
        import java.lang.reflect.Method;
        import java.lang.reflect.Proxy;

        public class Main{
            public static void main(String[] args){
                InvocationHandler handler = new InvocationHandler(){
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable{
                        System.out.println(method);
                        if(method.getName().equals("morning")){
                            System.out.println("Good morning, " + args[0]);
                        }
                        return null;
                    }
                };
                Hello hello = (Hello) Proxy.newProxyInstance(
                    Hello.class.getClassLoader(),
                    new Class[] {Hello.class},
                    handler);
                hello.morning("Bob");
            }
        }
    动态代理是JDK在运行期动态创建class字节码并加载的过程





    