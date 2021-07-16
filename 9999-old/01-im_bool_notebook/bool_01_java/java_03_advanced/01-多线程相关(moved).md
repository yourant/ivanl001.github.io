[toc]



## 1, 基本概念

### 1.1, 并发和并行

```shell
并发：concurrency，指多个线程在一个cpu的core进行交替的运行，从而实现一种同时在跑多个线程的需求

并行：parallelism，指对于多个cpu的core上，每个core上跑一个任务，实现同时跑多个任务的需求
```





## 2, 多线程创建的几种方式

### 2.1,  继承Thread类，实现run方法

```java
class FirstThread extends Thread {

    @Override
    public void run() {
        super.run();
        System.out.println(Thread.currentThread());
    }
}


// 这里是每逢100就创建一个新的线程
//第一种方式：继承Thread，然后创建继承后的类
for (int i = 0; i < 1000; i++) {
    if (i % 100 == 0) {
        FirstThread firstThread = new FirstThread();
        firstThread.start();
        //因为我们一般创建线程是为了做某些事情，所以需要继承Thread，把我们的事情封装在run方法内
        Thread thread = new Thread();
        thread.start();
    }
}
```



### 2.2, 实现Runnable接口

#### 实现Runnable，传递实现类到Thread中

```java
class SecondThread implements Runnable {

    @Override
    public void run() {
        System.out.println(Thread.currentThread());
    }
}


//第二种方式：实现Runnable接口，然后创建Thread类，把实现类传递到Thread中
for (int i = 0; i < 100; i++) {
    if (i % 10 == 0) {
        SecondThread secondThread = new SecondThread();
        //这里提示说不能单独创建一个线程，需要使用线程池，说的很对
        Thread thread = new Thread(secondThread);
        thread.start();
    }
}


```





#### 直接传递接口到Thread类中

```java
//第二种方式拓展, 这种方式是第二种方式实现接口的变形，直接继承，然后通过lamda表达式写出来
for (int i = 0; i < 100; i++) {
    if (i % 10 == 0) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread());
            }
        });
    }
}
```



#### 使用lambda表达式替换Runnable接口传递到Thread类中

```java
//第二种方式拓展, 这种方式是第二种方式实现接口的变形，直接继承，然后通过lamda表达式写出来
for (int i = 0; i < 100; i++) {
    if (i % 10 == 0) {
        new Thread(() -> {
            System.out.println(Thread.currentThread());
        });
    }
} 

```

> lambda表达式的一个记忆准则：

```java
new Thread(new Runnable() {
    @Override
    public void run() {
    }
});

//lambda表达式：拷贝小括号，写死右箭头，落地大括号
new Thread(() -> {});
```





### 2.3, 实现Callable方式

> Callable和Runnable类似，可以简单理解成增强版，因为Callable接口支持返回值和异常处理
>
> 但是Callable接口实现类不能直接传递给Thread，需要通过FutureTask(Future接口的实现类)来实现传递到Thread中去



```java
/**
 * 可以看作Runnable的升级，可以有返回值，可以抛异常
 */
class ThirdThread implements Callable {

    @Override
    public Object call() throws Exception {
        System.out.println(Thread.currentThread());
        return 100;
    }
}

//第三种方式：继承Callable
for (int i = 0; i < 100; i++) {
    if (i % 10 == 0) {
        ThirdThread thirdThread = new ThirdThread();
        //Callable的实现类并不能直接作为参数传递到Thread中，但是可以通过Future接口，FutureTask实现来传递
        FutureTask futureTask = new FutureTask(thirdThread);
        Thread thread = new Thread(futureTask);
        thread.start();
        System.out.println("Callable的返回值：" + futureTask.get());
    }
}
```



* 变形

```java
for (int i = 0; i < 100; i++) {
    if (i % 10 == 0) {
        ThirdThread thirdThread = new ThirdThread();

      	//直接传递接口进来， 注意：FutureTask如果不创建实例，直接传递到Thread中，那么就拿不到返回值了哈
        FutureTask futureTask = new FutureTask(new Callable() {
            @Override
            public Object call() throws Exception {
                System.out.println(Thread.currentThread());
                return 100;
            }
        });

        //Callable的实现类并不能直接作为参数传递到Thread中，但是可以通过Future接口，FutureTask实现来传递
        Thread thread = new Thread(futureTask);
        thread.start();
        System.out.println("Callable的返回值：" + futureTask.get());
    }
}
```





## 3, 线程池的使用

### 01, 为什么要用线程池

```java
class TheThread implements Runnable {
    @Override
    public void run() {
        System.out.println(Thread.currentThread());
    }
}

//这里相当于创建了100个线程池，而每个线程池的任务又相对简单。
//这对于系统来说，是比较重的负担，因为创建线程不是一个很简单的事情，所以可以通过创建线程池，用完之后回收，然后执行下次任务，这样做才会比较合适
for (int i = 0; i < 100; i++) {
    Thread thread = new Thread(new TheThread());
    thread.start();
}
```



### 02, 线程池

* 这种方式现在不太推荐

```java
//这里提示最好自己创建线程是，而不要用工厂Executors来创建，因为其内部有一个队列，其最大值是整形的最大值，队列卡住的时候可能会造成OOM
ExecutorService executorService = Executors.newFixedThreadPool(10);
for (int i = 0; i < 100; i++) {
    executorService.submit(new TheThread());
}
executorService.shutdown();

// 不推荐的原因，因为下面LinkedBlockingQueue的最大值可能是this(Integer.MAX_VALUE);在线程堵塞的时候可能会造成oom
public static ExecutorService newFixedThreadPool(int nThreads) {
  return new ThreadPoolExecutor(nThreads, nThreads,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>());
}

```



* ScheduledThreadPoolExecutor

```java
<!-- https://mvnrepository.com/artifact/org.apache.commons/commons-lang3 -->
<dependency>
  <groupId>org.apache.commons</groupId>
  <artifactId>commons-lang3</artifactId>
  <version>3.10</version>
</dependency>
  
// BasicThreadFactory是commons-lang3中的一个类


//推荐使用的第一种方式
ScheduledExecutorService executorService = new ScheduledThreadPoolExecutor(3,
        new BasicThreadFactory.Builder().namingPattern("example-schedule-pool-%d").daemon(true).build());
for (int i = 0; i < 100; i++) {
    executorService.schedule(new TheThread(), 0, TimeUnit.DAYS);
}
executorService.shutdown();
```



* ThreadPoolExecutor

```java
// 这种不行，可能会造成oom
ExecutorService executorService1 = Executors.newFixedThreadPool(10);

// 这种可以，已经指定了ArrayBlockingQueue的容量
ExecutorService executor = new ThreadPoolExecutor(10, 10,
                                                  60L, TimeUnit.SECONDS,
                                                  new ArrayBlockingQueue(10));

//下面这种需要添加依赖：
<!-- https://mvnrepository.com/artifact/com.google.guava/guava -->
<dependency>
  <groupId>com.google.guava</groupId>
  <artifactId>guava</artifactId>
  <version>27.0-jre</version>
</dependency>

//这种也可以，推荐使用的这种
ThreadFactory namedThreadFactory = new ThreadFactoryBuilder()
  .setNameFormat("demo-pool-%d").build();

//Common Thread Pool
ExecutorService pool = new ThreadPoolExecutor(5, 200,
                                              0L, TimeUnit.MILLISECONDS,
                                              new LinkedBlockingQueue<Runnable>(1024), namedThreadFactory, new ThreadPoolExecutor.AbortPolicy());

for (int i = 0; i < 100; i++) {
  pool.execute(new TheThread());
}
pool.shutdown();//gracefully shutdown
```













## 3, 集合类-线程不安全

### 3.1, 不安全演示

* List, ArrayList是线程不安全的， 在多线程中使用可能会报错：

```shell
java.util.ConcurrentModificationException
```

* 比如下面这段代码：

```java
List<String> list = new ArrayList<>();

for (int i = 0; i < 30; i++) {
    new Thread(() -> {
        list.add(UUID.randomUUID().toString().substring(0, 8));
        System.out.println(list);
    }, Integer.toString(i)).start();
}
```



### 3.2, 第一种解决办法：Vector

```java
List<String> list = new Vector<>();

for (int i = 0; i < 30; i++) {
    new Thread(() -> {
        list.add(UUID.randomUUID().toString().substring(0, 8));
        System.out.println(list);
    }, Integer.toString(i)).start();
}
```





### 3.3, 第二种解决办法：Collections

> ```shell
> 至于synchronizedList的内部锁，那是在并发执行add/remove的时候，不要把多个线程的东西加到list内部实现的同一个位置上去，导致数据丢失或者脏数据等问题，这是为了保证这个List在执行add/remove时不会存在并发问题。
> 简而言之，这两个锁是不同层面上的并发问题。
> 所以，当我们对synchronizedList进行遍历的时候一定不要忘了，在外部也加上synchronized(list)，以保证线程安全。
> ```



#### 3.3.1, List-Collections.synchronizedList

```java
List<String> list = Collections.synchronizedList(new ArrayList<>());

for (int i = 0; i < 30; i++) {
    new Thread(() -> {
        list.add(UUID.randomUUID().toString().substring(0, 8));
        System.out.println(list);
    }, Integer.toString(i)).start();
}
```



#### 3.3.2, Set-Collections.synchronizedSet

```java
Set<String> set = Collections.synchronizedSet(new HashSet<String>());
for (int i = 0; i < 30; i++) {
    new Thread(() -> {
        set.add(UUID.randomUUID().toString().substring(0, 8));
        System.out.println(set);
    }, Integer.toString(i)).start();
}
```



#### 3.3.3, Map-Collections.synchronizedMap

```java
Map<String, String> map = Collections.synchronizedMap(new HashMap<>());
for (int i = 0; i < 30; i++) {
    new Thread(() -> {
        map.put(UUID.randomUUID().toString().substring(0, 8), "zhang");
        System.out.println(map);
    }, Integer.toString(i)).start();
}
```





### 3.4, 第三种解决办法：JUC: java util concurrent

#### 3.4.1, List-CopyOnWriteArrayList

```java
//写时复制, 也就是读写分离思想
List<String> list = new CopyOnWriteArrayList<>();

for (int i = 0; i < 30; i++) {
    new Thread(() -> {
        list.add(UUID.randomUUID().toString().substring(0, 8));
        System.out.println(list);
    }, Integer.toString(i)).start();
}
```



#### 3.4.2, Set-CopyOnWriteArraySet

```java
//写时复制--set--CopyOnWriteArraySet
Set<String> set = new CopyOnWriteArraySet<>();
for (int i = 0; i < 30; i++) {
    new Thread(() -> {
        set.add(UUID.randomUUID().toString().substring(0, 8));
        System.out.println(set);
    }, Integer.toString(i)).start();
}
```



#### 3.4.3, Map-ConcurrentHashMap

```java
//写时复制--map--ConcurrentHashMap
Map<String, String> map = new ConcurrentHashMap<>();
for (int i = 0; i < 30; i++) {
    int finalI = i;
    new Thread(() -> {
        map.put(Integer.toString(finalI), UUID.randomUUID().toString().substring(0, 8));
        System.out.println(map  );
    }, Integer.toString(i)).start();
}
```







