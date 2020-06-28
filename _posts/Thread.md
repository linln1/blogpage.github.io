---
layout: post
---

摘要
<!--more-->
<!-- CreateTime:2020/6/27 12:09:27 -->
<div id="toc"></div>

#Java笔记
##Thread 

多线程模型是Java程序最基本的并发模型；
后续读写网络、数据库、Web开发等都依赖Java多线程模型
###
    Thread t = new Thread(()->{System.out.println("start new thread!");});
    t.start();

###wait && notify
    在synchronized块中才能调用wait()方法
    eg:
    class TaskQueue{
        Queue<String> queue = new LinkedList<>();

        public synchronized void addTask(String s){
            this.queue.add(s);
            this.notify();
        }

        public synchronized String getTask(){
            while (queue.isEmpty()){
                //释放this锁
                this.wait();
                //重获this锁
            }
            return queue.remove();
        }
    }
    当一个线程在this.wait()等待时，它就会释放this锁，从而使得其他线程能够在addTask()方法获得this锁
    第二个问题：如何让等待的线程被重新唤醒，然后从wait()方法返回？答案是在相同的锁对象上调用notify()方法

    import java.util.*;

    public static void main(String args[]) throws InterruptedExceptioin{
        var q = new TaskQueue<>();
        var ts = new ArrayList<Thread>();
        for(int i=0; i<5; i++){
            var t = new Thread(){
                public void run(){
                    while(true){
                        try{
                            String s = q.getTask();
                            System.out.println("execute task:" +s);
                        } catch(InterruptException e){
                            return;
                        }
                    }
                }
            };
            t.start();
            ts.add(t);
        }
        var add = new Thread(() -> {
            for (int i=0; i<10; i++) {
                // 放入task:
                String s = "t-" + Math.random();
                System.out.println("add task: " + s);
                q.addTask(s);
                try { Thread.sleep(100); } catch(InterruptedException e) {}
            }
        });
        add.start();
        add.join();
        Thread.sleep(100);
        for (var t : ts) {
            t.interrupt();
        }
    }

###ReentrantLock
    import java.util.concurrent

    public class Counter{
        private int count;

        public void add(int n){
            synchronized(this){
                count += n;
            }
        }
    }
    改造成
    public class Counter{
        private final Lock lock = new ReentrantLock();
        private int count;

        public void add(int n){
            lock.lock();
            try{
                count += n;
            }finally{
                lock.unlock();
            }
        }
    }
    //使用ReentrantLock比直接使用synchronized更安全，线程在tryLock()失败的时候不会导致死锁。
    ReentrantLock是Java代码实现的锁，我们就必须先获取锁，然后在finally中正确释放锁
    if (lock.tryLock(1, TimeUnit.SECONDS)) {
        try {
            ...
        } finally {
            lock.unlock();
        }
    }

###Condition
    synchronized -------wait && notify
    ReentrantLock -------Condition

    class TaskQueue {
        private final Lock lock = new ReentrantLock();
        private final Condition condition = lock.newCondition();
        private Queue<String> queue = new LinkedList<>();

        public void addTask(String s) {
            lock.lock();
            try {
                queue.add(s);
                condition.signalAll();
            } finally {
                lock.unlock();
            }
        }

        public String getTask() {
            lock.lock();
            try {
                while (queue.isEmpty()) {
                    condition.await();
                }
                return queue.remove();
            } finally {
                lock.unlock();
            }
        }
    }

    if (condition.await(1, TimeUnit.SECOND)) {
        // 被其他线程唤醒
    } else {
        // 指定时间内没有被其他线程唤醒
    }
    和tryLock()类似，await()可以在等待指定时间后，如果还没有被其他线程通过signal()或signalAll()唤醒，可以自己醒来

    我们想要的是：允许多个线程同时读，但只要有一个线程在写，其他线程就必须等待：

        读	    写
    读	允许	不允许
    写	不允许	不允许
    
    使用ReadWriteLock可以解决这个问题，它保证：

        只允许一个线程写入（其他线程既不能写入也不能读取）；
        没有写入时，多个线程允许同时读（提高性能）。
    
    public class Counter{
        private final ReadWriteLock rwlock = new ReentrantReadWriteLock();
        private final Lock rlock = rwlock.readLock();
        private fianl Lock wlock = rwlock.writeLock();
        private int[] counts = new int[10];

        public void inc(int index){
            wlock.lock();//加写锁
            try{
                counts[index] += 1;
            }finally{
                wlock.unlock();//释放写锁
            }
        }

        public int[] get{
            rlock.lock();
            try{
                return Arrays.copyOf(counts, counts.length);
            } finally{
                rlock.unlock();
            }
        }
    }
    适用条件是同一个数据，有大量线程读取，但仅有少数线程修改。

###StampedLock
    ReadWriteLock有个潜在的问题：如果有线程正在读，写线程需要等待读线程释放锁后才能获取写锁，即读的过程中不允许写，这是一种悲观的读锁。
    要进一步提升并发执行效率，Java 8引入了新的读写锁：StampedLock是一种乐观锁
    乐观地估计读的过程中大概率不会有写入，因此被称为乐观锁


###Concurrent
    BlockingQueue的意思就是说，当一个线程调用这个TaskQueue的getTask()方法时，该方法内部可能会让线程变成等待状态，直到队列条件满足不为空，线程被唤醒后，getTask()方法才会返回
    BlockingQueue非常有用，所以我们不必自己编写，可以直接使用Java标准库的java.util.concurrent包提供的线程安全的集合：ArrayBlockingQueue

###Atomic
    java.util.concurrent除了提供底层锁、并发集合外，还提供了一组原子操作的封装类，它们位于java.util.concurrent.atomic包

    AtomicInteger
        增加值并返回新值：int addAndGet(int delta)
        加1后返回新值：int incrementAndGet()
        获取当前值：int get()
        用CAS方式设置：int compareAndSet(int expect, int update)

    Atomic类是通过无锁（lock-free）的方式实现的线程安全（thread-safe）访问。它的主要原理是利用了CAS：Compare and Set
###ThreadPool
    Java标准库提供了ExecutorService接口表示线程池，它的典型用法如下
    // 创建固定大小的线程池:
    ExecutorService executor = Executors.newFixedThreadPool(3);
    // 提交任务:
    executor.submit(task1);
    executor.submit(task2);

    ExecutorService只是接口，Java标准库提供的几个常用实现类有：

    FixedThreadPool：线程数固定的线程池；
    CachedThreadPool：线程数根据任务动态调整的线程池；
    SingleThreadExecutor：仅单线程执行的线程池。

    创建这些线程池的方法都被封装到Executors这个类中

    import java.util.concurrent.*;

    public class Main{
        public static void main(String[] args){
            ExecutorService es = Executors.newFixedThreadPool(4);
            for(int i=0; i<6 ;i++){
                es.submit(new Task("" + i));
            }
            es.shutdown();
        }
    }
        
    class Task implements Runnable {
        private final String name;

        public Task(String name) {
            this.name = name;
        }

        @Override
        public void run() {
            System.out.println("start task " + name);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
            }
            System.out.println("end task " + name);
        }
    }

    有一种任务，需要定期反复执行，例如，每秒刷新证券价格。这种任务本身固定，需要反复执行的，可以使用ScheduledThreadPool。放入ScheduledThreadPool的任务可以定期反复执行
    ScheduledExecutorService ses = Executors.newScheduledThreadPool(4);


    当我们提交一个Callable任务后，我们会同时获得一个Future对象，然后，我们在主线程某个时刻调用Future对象的get()方法，就可以获得异步执行的结果。在调用get()时，如果异步任务已经完成，我们就直接获得结果。如果异步任务还没有完成，那么get()会阻塞，直到任务完成后才返回结果。
    ExecutorService executor = Executors.newFixedThreadPool(4); 
    // 定义任务:
    Callable<String> task = new Task();
    // 提交任务并获得Future:
    Future<String> future = executor.submit(task);
    // 从Future获取异步执行返回的结果:
    String result = future.get(); // 可能阻塞

    一个Future<V>接口表示一个未来可能会返回的结果，它定义的方法有：

        get()：获取结果（可能会等待）
        get(long timeout, TimeUnit unit)：获取结果，但只等待指定的时间；
        cancel(boolean mayInterruptIfRunning)：取消当前任务；
        isDone()：判断任务是否已完成。

###ForkJoin
    Fork/Join线程池，它可以执行一种特殊的任务：把一个大任务拆成多个小任务并行执行
    使用Fork/Join对大数据进行并行求和:

    import java.util.Random;
    import java.util.concurrent.*;

    public class Main {
    public static void main(String[] args) throws Exception {
        // 创建2000个随机数组成的数组:
        long[] array = new long[2000];
        long expectedSum = 0;
        for (int i = 0; i < array.length; i++) {
            array[i] = random();
            expectedSum += array[i];
        }
        System.out.println("Expected sum: " + expectedSum);
        // fork/join:
        ForkJoinTask<Long> task = new SumTask(array, 0, array.length);
        long startTime = System.currentTimeMillis();
        Long result = ForkJoinPool.commonPool().invoke(task);
        long endTime = System.currentTimeMillis();
        System.out.println("Fork/join sum: " + result + " in " + (endTime - startTime) + " ms.");
    }

    static Random random = new Random(0);

    static long random() {
        return random.nextInt(10000);
    }
}

class SumTask extends RecursiveTask<Long> {
    static final int THRESHOLD = 500;
    long[] array;
    int start;
    int end;

    SumTask(long[] array, int start, int end) {
        this.array = array;
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        if (end - start <= THRESHOLD) {
            // 如果任务足够小,直接计算:
            long sum = 0;
            for (int i = start; i < end; i++) {
                sum += this.array[i];
                // 故意放慢计算速度:
                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                }
            }
            return sum;
        }
        // 任务太大,一分为二:
        int middle = (end + start) / 2;
        System.out.println(String.format("split %d~%d ==> %d~%d, %d~%d", start, end, start, middle, middle, end));
        SumTask subtask1 = new SumTask(this.array, start, middle);
        SumTask subtask2 = new SumTask(this.array, middle, end);
        invokeAll(subtask1, subtask2);
        Long subresult1 = subtask1.join();
        Long subresult2 = subtask2.join();
        Long result = subresult1 + subresult2;
        System.out.println("result = " + subresult1 + " + " + subresult2 + " ==> " + result);
        return result;
    }
}

###ThreadLoacl
    Java标准库提供了一个特殊的ThreadLocal，它可以在一个线程中传递同一个对象。
    ThreadLocal实例通常总是以静态字段初始化如下：

    static ThreadLocal<User> threadLocalUser = new ThreadLocal<>();

    典型使用方式如下：

    void processUser(user) {
        try {
            threadLocalUser.set(user);
            step1();
            step2();
        } finally {
            threadLocalUser.remove();
        }
    }
    void step1() {
        User u = threadLocalUser.get();
        log();
        printUser();
    }

    实际上，可以把ThreadLocal看成一个全局Map<Thread, Object>：每个线程获取ThreadLocal变量时，总是使用Thread自身作为key：

    Object threadLocalValue = threadLocalMap.get(Thread.currentThread());
    因此，ThreadLocal相当于给每个线程都开辟了一个独立的存储空间，各个线程的ThreadLocal关联的实例互不干扰。

    最后，特别注意ThreadLocal一定要在finally中清除：
