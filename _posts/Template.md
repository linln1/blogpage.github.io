---
layout: post
---

摘要
<!--more-->
<!-- CreateTime:2020/6/24 21:21:21 -->


<div id="toc"></div>

#Java笔记
##Templates

###ArrayList//可变长度数组
    内部是Object[] array
    所以比如
    ArrayList list = new ArrayList();
    list.add("add");
    必须要强制类型转换
    String first = (String) list.get(0);

    模板 
    public class ArrayList<T>{
       private T[] array;
    }
    ArrayList<String> list = new ArrayList<String>();
    向上转型
    ArrayList<T> implements List<T>
    List<String> list = new ArrayList<String>();

###Usage
    接口中也可以使用泛型
    import java.util.Arrays;
    Arrays.sort(Object[])
    public interface Comparable<T>{
        int CompareTo(T o);
    }
    class A implements Comparable<A>{
        public int CompareTo(A other){
            return this.name.CompareTo(other.name);
        }
    }

###Coding
    ####泛型T不能用在静态方法
    eg:
    1)Wrong
    public static Pair<T> create(T.first, T last){
        return new Pair<T>(first, last);
    }
    2)Correct
    public static <K> Pair<K> create(K.first, K.last){
        return new Pair<K>(first, last);
    }
 
    泛型还可多样
    public class Pair<T,K>{}

###擦拭法
    编译器把<T>看做Object,只有在运行的时候强制转型
    1)所以<T>不能是基本类型比如int
    2)无法取得带有泛型的class
    3)无法判断带泛型的类型
    4)不能实例化T, private T first = new T();
        只能
        public Pair(Class<T> class){
            first = class.newInstance();
            last = class.newInstance();
        }
    ####泛型继承
    public class IntPart extends Pair<Integer>{}
    子类可以获得父类的泛型类型
    import java.lang.reflect.ParameterizedType;
    import java.lang.reflect.Type;

    public class Main{
        public static void main(String[] args){
            Class<IntPart> clazz = IntPair.class;
            Type t = clazz.getGenericSuperclass();
            if( t instanceof ParameterizedType){
                ParameterizedType pt = ()t;

            }
        }
    }

###Extends
    get方法<? extends Number> 上界是Number,他和它的子类都可以
    set方法<? extends Number> 是个废物,传进去的参数只能是null

    int sumOfList(List<? extends Integer> list){
        int sum = 0;
        for (int i = 0 ; i < list.size() ; i++ ){
            Integer n = list.get(i);
            sum += n;
        }
        return sum;
    }

    1)这样实现的sumOfList只读List,不可以修改,除非恶意set(null)
    2)还可以限制类型范围<T extends Numbers>

###Super
    set方法<? super Integer> 下界是Integer,它和它的父类都可以
    get方法<? super Integer> 左侧只能用Object接受
    
    1)内部代码对于参数只能写不能读

    eg:
    public class Collections{
        public static <T> void copy(List<? super T>dest, List<? extends T>src){
            for(int i=0; i<src.size();i++){
                T t = src.get(i);
                dest.add(t);
            }
        }
    }

    2)Pair<?> 不能读不能写，只能做一些null判断,Object引用

###泛型与反射
    部分反射API也是泛型
    Class<T>
    getSuperClass() --> Class<? super T>
    Constructor<T>

    Pair<String>[] ps = new Pair<String>[2];
    Pair<String>[] ps = (Pair<String>[]) new Pair[2];

    @SuppressWarnings("unchecked");

