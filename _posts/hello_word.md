---
layout: post
---

摘要
<!--more-->
<!-- CreateTime:2020/6/24 10:51:37 -->


<div id="toc"></div>
#Hello world!

#Java笔记
##Java核心类

###StringBuilder
    1)为了能高效拼接字符串，Java标准库提供了StringBuilder，它是一个可变对象，可以预分配缓冲区，这样，往StringBuilder中新增字符时，不会创建新的临时对象

    eg:
        StringBuilder sb = new StringBuilder(1024);
        for(int i = 0; i < 1000; i++){
            sb.append(',');
            sb.append(i);
            <!-- 原本需要 s = s + "," + i ;-->
        }

        String s = sb.toString();
    
    2)定义的append()方法会返回this
    故可以进行链式操作

    eg:
        public class Main{
            public static void main(String[] args){
                var sb = new StringBuilder(1024);
                sb.append("Mr ")
                  .append("Bob")
                  .insert(0, "Hello, ");
                System.out.println(sb.toString());
            }
        }

//设计一个支持链式操作的类

    class Adder{
        private int sum = 0;
        public Adder add (int n){
            sum += n;
            return this;
        }

        public Adder inc (){
            sum++;
            return this;
        }

        public int value(){
            return sum;
        }
    }
    Adder a = new Adder();
    a.add(1).inc().add(2);

//普通的字符串+操作，编译器自动把多个连续的+编码为StringConcatFactory,优化为StringBuilder

    //构造一个INSERT操作
    static String builderInsertSql(String table, String[] fields){
        var ins = new StringBuilder();
        ins.append("INSERT TO ").append(table).append(" (");
        for(int i = 0; i < fileds.length() - 1; i++){
            ins.append(fileds[i]).append(", ");
        }
        ins.append(fields[-1]).append(") VALUES ").append("(");
        for(int i = 0; i < fileds.length() - 1; i++){
            ins.append("?").append(", ");
        ins.append("?)");
        }
        String ans = ins.toString();
        return ans;
    }

###StringJoiner  扩展StringBuilder的功能，可以用分隔符连接数组
    eg:
    var sj = new StringJoiner(", ");
    for(String name: names)
        sj.add(name);
    //要加一个开头和一个结尾
    eg:
    var sh = new StringJoiner(", ", "Hello ", "!");
    //join()
    var st = String.join(", ", names);

###包装类型
    java.lang.Boolean
    java.lang.Byte
    java.lang.Integer
    java.lang.Short
    java.lang.Long
    java.lang.Float
    java.lang.Bouble
    java.lang.Character

    用equals进行比较/不能用==
    Integer小的时候，等价于Integer x = Integer.ValueOf(127),会返回相同实例，会==，大的时候，即使value相同，也不==
    AutoBoxing
    Integer n = new Integer(100);
    Integer n = Integer.ValueOf(100);

    Boolean t = Boolean.TRUE;
    Boolean f = Boolean.FALSE;

    int max = Integer.MAX_VALUE;
    int min = Integer.MIN_VALUE;

    int sizeOfLong = Long.SIZE;
    int byteOfLong = Long.BYTES;

###JavaBean
    boolean字段读方法/写方法
        public boolean isChild()
        public void setChild(boolean value)

    BeanInfo info = Introspector.getBeanInfo(Person.class);
    for(PropertDescriptor pd: info.getPropertyDescriptors()){
        pd.getName();
        pd.getReadMethod();
        pd.getWriteMethod();
    }

###enumrator类
    public class Weekday{
        public static final int SUN = 0;
        ...
    }

    Weekday.SUN == day

    enum Weekday{
        SUN(0,"sunday"), MON(1), TUE, WED, THU, FRI, SAT;
    }
    Weekday day = Weekday.SUN;
    day.toString() // (sunday)
    day.name() // SUN

    引用类型比较用equals(),
    ==比较的是左右两端的变量是不是同一个对象

    enum继承自java.lang.Enum,且无法被继承
    没有new操作符

###记录类
    String, Integer 是不变类，不变类不能派生子类
        1)定义class用final,无法派生分类
        2)每个字段使用final,无法修改实例
    
    Java 14  CLass Record
    使用关键字record

    public record Point(int x, int y){}
    //自动，构造函数，toString(), equals(), hashCode()
    //可以用她来一行写出一个不变类
    eg:
    public record Point(int x, int y){
        public Point{
            if(x<0||y<0){
                throw new IllegalArgumentException();
            }
        }
        //构造静态方法
        public static Point of(){
            return new Point(0,0);
        }
        public static Point of(int x, int y){
            return new Point(x,y);
        }
    }

###BigInteger & BigDecimal
    //超过64为long的大整数
    java.math.BigInteger;
    //内部用一个int[]数组来模拟超大整数
    BigInteger bi = new BigInteger("1234567890");
    bi.pow(5);

    i1.add(i2);
    i1.longValue();

    BigDecimal用CompareTo来比较

###SecureRandom
    SecureRandom sr = new SecureRandom();
    System.out.println(sr.nextInt(1000));







