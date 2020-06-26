---
layout: post
---

摘要
<!--more-->
<!-- CreateTime:2020/6/25 10:30:21 -->


<div id="toc"></div>

#Java笔记
##Collections

###List
    import java.util.ArrayList;
    import java.util.List;
    List<String> list = new ArrayList<>();
    public clas Main{
        public static void main(String[] args){
            List<String> list = List.of("apple", "pear", "banana");
            for(int i=0;i<list.size();i++){
                String s = list.get(i);
                System.out.println(s);
            }
        } 
    }

    import java.util.Iterator;
    import java.util.List;

    public class Main{
        for(Iterator<String> it = list.iterator(); it.hasNext(); ) 
    //  for(String s : list){
            System.out.println(s);
        }
    }

    import java.util.List;
    List<String> list = List.of("apple", "pear", "banana");
    Object[] array = list.toArray();
    Integer[] array = list.toArray(new Integer[3]);
    for(Object s : array){
        System.out.println(s);
    }

    Integer[] array = list.toArray(new Integer[list.size()]);
    Integer[] array = list.toArray(Integer[]:: new);

###equals
    public boolean equals(Object o){
        if(Object instanceof Person){
            Person p = (Person) o;
            boolean nameEquals = false;
            if(this.name == null && p.name == null){
                nameEquals = true;
            }
            if(this.name !=null){
                nameEquals = this.name.equals(p.name);
            }
            return nameEquals && this.age == p.age;
        }
        return false;
    }

###Map
    <key,value>通过key告诉高速查找value
    import java.util.HashMap;
    import java.util.Map;
    Student s = new Student("Xiao Ming, 19);
    Map<String, Student> map = new HashMap<>();
    map.put = ("Xiao Ming", s);
    Student target = map.get("Xiao Ming");
    
    for(String key: map.keySet()){
        Integer value = map.get(key);
    }

###equals && hashCode
    public class Person{
        String firstName;
        String lastName;
        int age;

        @Override
        int hashCode(){
            int h=0;
            h = 31*h+firstName.hashCode();
        }
    }

###EnumMap TreeMap
    sortedMap 有序， 但它只是一个接口，他的实现类是TreeMap
    Map<Person, Integer> map = new TreeMap<>(new Comparator<Person>(){
        public int compare(Person p1, Person p2){
            return p1.name.compareTo(p2.name);
        }
    })

###Properties
    提供了一个Properties来表示配置 Map<String, String>,但本质上他是一个Hashtable，
    使用Properties读取配置文件，Java配置文件默认以.properties为拓展名，每行以key=value表示，以#开头的是注释

    #setting.properties

    last_open_file = /data/hello.txt
    auto_save_interval = 50

    从文件系统读取这个.properties
    Properties props = new Properties()
    props.load(new java.io.FileInputStream(f));

    String filepath = props.getProperty("last_open_file");
    String interval = props.getProperty("auto_save_interval", "120")



###Set
    .add.remove.contains.size
    放入Set的元素和Map的key类似都要事先正确的equals(),hashCode()
    public class Hashset<E> implements Set<E>{
        private HashMap<E, Object> map = new HashMap<>();
        
        private static final Object PRESENT = new Object();

        public boolean add(E e){
            return map.put(e, PRESENT) == null;
        }

        public boolean contains(Object o){
            return map.containsKey(o);
        }

        public boolean remove(Object o){
            return map.remove(o) == PRESENT;
        }
    }    

###Queue
    poll() remove() peek() add() offer()
    List<String> list = new LinkedList<>();
    Queue<String> queue = mew LinkedList<>();
###PriorityQueue
    Queue<String> queue = new PriorityQueue<>(
        new UserComparator()
    );

    class UserComparator implements Comparator<User>{
        public int compare(User u1, User u2){
            if(u1.number.charAt(0) == u2.number.charAt(0)){
                return u1.number.compareTo(u2.number);
            }
            if(u1.number.charAt(0) == 'V'){
                return -1
            }else{
                return 1;
            }
        }
    }

###Deque
    addLast().offerLast().removeFirst.pollFirst().getFirst().peekFirst().addFirst().offerFirst().removeLast().pollLast().getLast().peekLast()
    Deque是一个接口，他的实现类有ArrayDeque && LinkedList
    
###Stack
    用Deque来实现的，没有特定的实现类

###Iterator
    Iterator<String> it = list.iterator(); it.hasNext();

    class ReverseList<T> implements Iterable<T>{
        private List<T> list = new ArrayList<>();

        public void add(T t){
            list.add(t);
        }

        @Override
        public Iterator<T> iterator(){
            return new ReverseIterator(list.size());
        }

        class ReverseIterator implements Iterator<T>{
            int index;

            ReverseIterator(int index){
                this.index = index;
            }

            @Override
            public boolean hasNext(){
                return index > 0;
            }

            @Override
            public T next(){
                index --;
                return ReverseList.this.list.get(index);
            }
            
        }

    }

###Collections
    import java.util.collections
    提供了一系列的静态方法，能更方便地操纵各种集合

    public static boolean addAll(Collection<? super T>c,T... elements){...}

    不可变集合
    import java.util.*;
    List<T> unmodifiableList(List<? extends T> list)
    Set<T> unmodifiableSet(Set<? extends T> set)
    Map<K, V> unmodifiableMap(Map<? extends K, ? extends V> m)

    


    