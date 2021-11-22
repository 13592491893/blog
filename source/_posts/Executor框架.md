---
abbrlink: 54338
title: Executor框架
comments: true
toc: true
description: Executor框架
top_img: https://gitee.com/gsshy/picgo/raw/master/img/top.jpg
categories:
  - java
tags:
  - Executor
  - 多线程
  - 线程
date: 2021-2-21 16:00:00

---
# Executor框架

## 一、Executor、Executors、ExecutorService

在Java 5之后，并发编程引入了一堆新的启动、调度和管理线程的API。Executor框架便是Java 5中引入的，其内部使用了线程池机制，它在java.util.cocurrent 包下，通过该框架来控制线程的启动、执行和关闭，可以简化并发编程的操作。因此，在Java 5之后，通过Executor来启动线程比使用Thread的start方法更好，除了更易管理，效率更好（用线程池实现，节约开销）外，还有关键的一点：有助于避免this逃逸问题——如果我们在构造器中启动一个线程，因为另一个任务可能会在构造器结束之前开始执行，此时可能会访问到初始化了一半的对象用Executor在构造器中。Eexecutor作为灵活且强大的异步执行框架，其支持多种不同类型的任务执行策略，提供了一种标准的方法将任务的提交过程和执行过程解耦开发，基于生产者-消费者模式，其提交任务的线程相当于生产者，执行任务的线程相当于消费者，并用Runnable来表示任务，Executor的实现还提供了对生命周期的支持，以及统计信息收集，应用程序管理机制和性能监视等机制。

### 1、Executor的UML图：（常用的几个接口和子类）

![image-20210323103723142](https://gitee.com/gsshy/picgo/raw/master/img/image-20210323103723142.png)

Executor框架包括：线程池，Executor，Executors，ExecutorService，CompletionService，Future，Callable等。

### 2、Executor和ExecutorService

**Executor：**一个接口，其定义了一个接收Runnable对象的方法executor，其方法签名为executor(Runnable command),该方法接收一个Runable实例，它用来执行一个任务，任务即一个实现了Runnable接口的类，一般来说，Runnable任务开辟在新线程中的使用方法为：new Thread(new RunnableTask())).start()，但在Executor中，可以使用Executor而不用显示地创建线程：executor.execute(new RunnableTask()); // 异步执行

ExecutorService：是一个比Executor使用更广泛的子类接口，其提供了生命周期管理的方法，返回 Future 对象，以及可跟踪一个或多个异步任务执行状况返回Future的方法；可以调用ExecutorService的shutdown（）方法来平滑地关闭 ExecutorService，调用该方法后，将导致ExecutorService停止接受任何新的任务且等待已经提交的任务执行完成(已经提交的任务会分两类：一类是已经在执行的，另一类是还没有开始执行的)，当所有已经提交的任务执行完毕后将会关闭ExecutorService。因此我们一般用该接口来实现和管理多线程。

通过 ExecutorService.submit() 方法返回的 Future 对象，可以调用isDone（）方法查询Future是否已经完成。当任务完成时，它具有一个结果，你可以调用get()方法来获取该结果。你也可以不用isDone（）进行检查就直接调用get()获取结果，在这种情况下，get()将阻塞，直至结果准备就绪，还可以取消任务的执行。Future 提供了 cancel() 方法用来取消执行 pending 中的任务。ExecutorService 部分代码如下：

```java
public interface ExecutorService extends Executor {
	void shutdown();
	<T> Future<T> submit(Callable<T> task);
	<T> Future<T> submit(Runnable task, T result);
	<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException;
}
```

### 3、Executors类： 主要用于提供线程池相关的操作

Executors类，提供了一系列工厂方法用于创建线程池，返回的线程池都实现了ExecutorService接口。

1. public static ExecutorService newFiexedThreadPool(int Threads) 创建固定数目线程的线程池。

2. public static ExecutorService newCachedThreadPool()：创建一个可缓存的线程池，调用execute 将重用以前构造的线程（如果线程可用）。如果没有可用的线程，则创建一个新线程并添加到池中。终止并从缓存中移除那些已有 60 秒钟未被使用的线程。

3. public static ExecutorService newSingleThreadExecutor()：创建一个单线程化的Executor。

4. public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize)：创建一个支持定时及周期性的任务执行的线程池，多数情况下可用来替代Timer类。

### 4、Executor VS ExecutorService VS Executors

正如上面所说，这三者均是 Executor 框架中的一部分。Java 开发者很有必要学习和理解他们，以便更高效的使用 Java 提供的不同类型的线程池。总结一下这三者间的区别，以便大家更好的理解：

Executor 和 ExecutorService 这两个接口主要的区别是：ExecutorService 接口继承了 Executor 接口，是 Executor 的子接口
Executor 和 ExecutorService 第二个区别是：Executor 接口定义了 execute()方法用来接收一个Runnable接口的对象，而 ExecutorService 接口中的 submit()方法可以接受Runnable和Callable接口的对象。
Executor 和 ExecutorService 接口第三个区别是 Executor 中的 execute() 方法不返回任何结果，而 ExecutorService 中的 submit()方法可以通过一个 Future 对象返回运算结果。
Executor 和 ExecutorService 接口第四个区别是除了允许客户端提交一个任务，ExecutorService 还提供用来控制线程池的方法。比如：调用 shutDown() 方法终止线程池。可以通过 《Java Concurrency in Practice》 一书了解更多关于关闭线程池和如何处理 pending 的任务的知识。
Executors 类提供工厂方法用来创建不同类型的线程池。比如: newSingleThreadExecutor() 创建一个只有一个线程的线程池，newFixedThreadPool(int numOfThreads)来创建固定线程数的线程池，newCachedThreadPool()可以根据需要创建新的线程，但如果已有线程是空闲的会重用已有线程。
下面给出一个Executor执行Callable任务的示例代码：

```java
import java.util.ArrayList;   
import java.util.List;   
import java.util.concurrent.*;   
  
public class CallableDemo{   
    public static void main(String[] args){   
        ExecutorService executorService = Executors.newCachedThreadPool();   
        List<Future<String>> resultList = new ArrayList<Future<String>>();   
        //创建10个任务并执行   
        for (int i = 0; i < 10; i++){   
            //使用ExecutorService执行Callable类型的任务，并将结果保存在future变量中   
            Future<String> future = executorService.submit(new TaskWithResult(i));   
            //将任务执行结果存储到List中   
            resultList.add(future);   
        }   
        //遍历任务的结果   
        for (Future<String> fs : resultList){   
                try{   
                    while(!fs.isDone);//Future返回如果没有完成，则一直循环等待，直到Future返回完成  
                    System.out.println(fs.get());     //打印各个线程（任务）执行的结果   
                }catch(InterruptedException e){   
                    e.printStackTrace();   
                }catch(ExecutionException e){   
                    e.printStackTrace();   
                }finally{   
                    //启动一次顺序关闭，执行以前提交的任务，但不接受新任务  
                    executorService.shutdown();   
                }   
        }   
    }   
}   
  
class TaskWithResult implements Callable<String>{   
    private int id;   
  
    public TaskWithResult(int id){   
        this.id = id;   
    }   
  
    /**  
     * 任务的具体过程，一旦任务传给ExecutorService的submit方法， 
     * 则该方法自动在一个线程上执行 
     */   
    public String call() throws Exception {  
        System.out.println("call()方法被自动调用！！！    " + Thread.currentThread().getName());   
        //该返回结果将被Future的get方法得到  
        return "call()方法被自动调用，任务返回的结果是：" + id + "    " + Thread.currentThread().getName();   
    }   
}  
```

### 5、自定义线程池

自定义线程池，可以用ThreadPool Executor类创建，它有多个构造方法来创建线程池，用该类很容易实现自定义的线程池，这里先贴上示例程序：

```java
import java.util.concurrent.ArrayBlockingQueue;   
import java.util.concurrent.BlockingQueue;   
import java.util.concurrent.ThreadPoolExecutor;   
import java.util.concurrent.TimeUnit;   
  
public class ThreadPoolTest{   
    public static void main(String[] args){   
        //创建等待队列   
        BlockingQueue<Runnable> bqueue = new ArrayBlockingQueue<Runnable>(20);   
        //创建线程池，池中保存的线程数为3，允许的最大线程数为5  
        ThreadPoolExecutor pool = new ThreadPoolExecutor(3,5,50,TimeUnit.MILLISECONDS,bqueue);   
        //创建七个任务   
        Runnable t1 = new MyThread();   
        Runnable t2 = new MyThread();   
        Runnable t3 = new MyThread();   
        Runnable t4 = new MyThread();   
        Runnable t5 = new MyThread();   
        Runnable t6 = new MyThread();   
        Runnable t7 = new MyThread();   
        //每个任务会在一个线程上执行  
        pool.execute(t1);   
        pool.execute(t2);   
        pool.execute(t3);   
        pool.execute(t4);   
        pool.execute(t5);   
        pool.execute(t6);   
        pool.execute(t7);   
        //关闭线程池   
        pool.shutdown();   
    }   
}   
  
class MyThread implements Runnable{   
    @Override   
    public void run(){   
        System.out.println(Thread.currentThread().getName() + "正在执行。。。");   
        try{   
            Thread.sleep(100);   
        }catch(InterruptedException e){   
            e.printStackTrace();   
        }   
    }   
}  
```

运行结果如下：

![image-20210323105726043](https://gitee.com/gsshy/picgo/raw/master/img/image-20210323105726043.png)

### 6、比较Executor和new Thread()

new Thread的弊端如下：

- 每次new Thread新建对象性能差。
- 线程缺乏统一管理，可能无限制新建线程，相互之间竞争，及可能占用过多系统资源导致死机或oom。
- 缺乏更多功能，如定时执行、定期执行、线程中断。

相比new Thread，Java提供的四种线程池的好处在于：

-  重用存在的线程，减少对象创建、消亡的开销，性能佳。
- 可有效控制最大并发线程数，提高系统资源的使用率，同时避免过多资源竞争，避免堵塞。
- 提供定时执行、定期执行、单线程、并发数控制等功能。

## 二、ExecutorService和ThreadPoolExecutor

### 1、创建线程池有两种方式

![image-20210323110453172](https://gitee.com/gsshy/picgo/raw/master/img/image-20210323110453172.png)

1. 通过ThreadPoolExecutor构造函数(推荐)

   ```java
   import java.util.concurrent.ArrayBlockingQueue;
   import java.util.concurrent.ThreadPoolExecutor;
   import java.util.concurrent.TimeUnit;
   
   public class ThreadPoolExecutorDemo {
   
       private static final int CORE_POOL_SIZE = 5;
       private static final int MAX_POOL_SIZE = 10;
       private static final int QUEUE_CAPACITY = 100;
       private static final Long KEEP_ALIVE_TIME = 1L;
       public static void main(String[] args) {
   
           //使用阿里巴巴推荐的创建线程池的方式
           //通过ThreadPoolExecutor构造函数自定义参数创建
           ThreadPoolExecutor executor = new ThreadPoolExecutor(
                   CORE_POOL_SIZE,
                   MAX_POOL_SIZE,
                   KEEP_ALIVE_TIME,
                   TimeUnit.SECONDS,
                   new ArrayBlockingQueue<>(QUEUE_CAPACITY),
                   new ThreadPoolExecutor.CallerRunsPolicy());
   
           for (int i = 0; i < 10; i++) {
               //创建WorkerThread对象（WorkerThread类实现了Runnable 接口）
               Runnable worker = new MyRunnable("" + i);
               //执行Runnable
               executor.execute(worker);
           }
           //终止线程池
           executor.shutdown();
           while (!executor.isTerminated()) {
           }
           System.out.println("Finished all threads");
       }
   }
   ```

   corePoolSize: 核心线程数为 5。
   maximumPoolSize ：最大线程数 10
   keepAliveTime : 等待时间为 1L。
   unit: 等待时间的单位为 TimeUnit.SECONDS。
   workQueue：任务队列为 ArrayBlockingQueue，并且容量为 100;
   handler:饱和策略为 CallerRunsPolicy。

   ```java
   output
   pool-1-thread-3 Start. Time = Sun Apr 12 11:14:37 CST 2020
   pool-1-thread-5 Start. Time = Sun Apr 12 11:14:37 CST 2020
   pool-1-thread-2 Start. Time = Sun Apr 12 11:14:37 CST 2020
   pool-1-thread-1 Start. Time = Sun Apr 12 11:14:37 CST 2020
   pool-1-thread-4 Start. Time = Sun Apr 12 11:14:37 CST 2020
   pool-1-thread-3 End. Time = Sun Apr 12 11:14:42 CST 2020
   pool-1-thread-4 End. Time = Sun Apr 12 11:14:42 CST 2020
   pool-1-thread-1 End. Time = Sun Apr 12 11:14:42 CST 2020
   pool-1-thread-5 End. Time = Sun Apr 12 11:14:42 CST 2020
   pool-1-thread-1 Start. Time = Sun Apr 12 11:14:42 CST 2020
   pool-1-thread-2 End. Time = Sun Apr 12 11:14:42 CST 2020
   pool-1-thread-5 Start. Time = Sun Apr 12 11:14:42 CST 2020
   pool-1-thread-4 Start. Time = Sun Apr 12 11:14:42 CST 2020
   pool-1-thread-3 Start. Time = Sun Apr 12 11:14:42 CST 2020
   pool-1-thread-2 Start. Time = Sun Apr 12 11:14:42 CST 2020
   pool-1-thread-1 End. Time = Sun Apr 12 11:14:47 CST 2020
   pool-1-thread-4 End. Time = Sun Apr 12 11:14:47 CST 2020
   pool-1-thread-5 End. Time = Sun Apr 12 11:14:47 CST 2020
   pool-1-thread-3 End. Time = Sun Apr 12 11:14:47 CST 2020
   pool-1-thread-2 End. Time = Sun Apr 12 11:14:47 CST 2020
   ```

2. 通过Executor框架的Executors创建

   Executors返回的是ExecutorService

   ![image-20210323111609960](https://gitee.com/gsshy/picgo/raw/master/img/image-20210323111609960.png)

### 2、ExecutorService

Java.util.concurrent.ExecutorService接口代表一种异步执行机制，它能够在后台执行任务。因此ExecutorService与thread pool是非常相似的。事实上，在java.util.package包中ExecutorService的具体实现就是一个线程池的具体实现。下面是一个简单的例子：

```java
ExecutorService executorService = Executors.newFixedThreadPool(10);
 
executorService.execute(new Runnable() {
    public void run() {
        System.out.println("Asynchronous task");
    }
});
 
executorService.shutdown();
```

首先，通过newFixedThreadPool()工厂方法创建一个ExecutorService的实例。这个方法创建了一个可以有10个线程执行任务的线程池。

第二，Runnable接口的匿名实现类作为参数被传递给execute()方法。Runable将会被ExecutorService中的一个线程来执行。

下面是委托任务给ExecutorService的一些不同的方式：

- execute(Runnable)
- submit(Runnable)
- submit(Callable)
- invokeAny(…)
- invokeAll(…)

下面来逐个看看这些方法。

- execute(Runnable)

execute(Runnable) 方法接受一个java.lang.Runable对象的实例，并异步执行之。下面是一个使用ExecutorService执行Runnable的例子：

```java
ExecutorService executorService = Executors.newSingleThreadExecutor();
 
executorService.execute(new Runnable() {
    public void run() {
        System.out.println("Asynchronous task");
    }
});
 
executorService.shutdown();
```

这种方式不能获得Runnable执行的结果，如果有这种需要，你将要使用Callable。

- submit(Runnable)

submit(Runnable) 方法也接收一个Runnable接口的具体实现，并返回一个Future对象。Future对象可以用来检测Runable是否执行完成。

```java
Future future = executorService.submit(new Runnable() {
    public void run() {
        System.out.println("Asynchronous task");
    }
});
 
future.get();  //returns null if the task has finished correctly.
```

- submit(Callable)

submit(Callable)方法与submit(Runnable)方法相似，除了接收的参数有所不同。Callable实例非常类似于Runnable,不同的是call方法可以返回一个结果，Runnable.run()方法不能返回一个结果(因为是void类型)，就算线程执行完了，成功了future.get()也只是得到null

可以通过submit(Callable)方法返回的Future对象获取Callable的结果。下面是一个使用Callable的例子：

```java
Future future = executorService.submit(new Callable(){
    public Object call() throws Exception {
        System.out.println("Asynchronous Callable");
        return "Callable Result";
    }
});
 
System.out.println("future.get() = " + future.get());
```

上面代码的输出结果是：

```java
Asynchronous Callable
future.get() = Callable Result
```

- invokeAny(…)

invokeAny()方法接收一个Callable对象或者Callable的子接口实例的集合作为参数，这个方法不会返回Future,但会返回集合中某一个Callable的结果。你不能确定你得到是哪个Callable的结果。只是已执行完成的Callable中的一个。

如果一个任务已经完成（或者抛出了异常），剩余的Callable任务将被取消。 
下面是示例代码：

```java
ExecutorService executorService = Executors.newSingleThreadExecutor();
 
Set<Callable<String>> callables = new HashSet<Callable<String>>();
 
callables.add(new Callable<String>() {
    public String call() throws Exception {
        return "Task 1";
    }
});
callables.add(new Callable<String>() {
    public String call() throws Exception {
        return "Task 2";
    }
});
callables.add(new Callable<String>() {
    public String call() throws Exception {
        return "Task 3";
    }
});
 
String result = executorService.invokeAny(callables);
 
System.out.println("result = " + result);
 
executorService.shutdown();
```

- 示例代码将会打印给定的Callable集合中一个Callable任务返回的结果。我尝试执行了多次，结果是变化的。有时候是“Task1”,有时候是“Task 2”等。
- 

- invokeAll(…)

invokeAll()接收一个Callable对象的集合作为参数，该方法会调用你传给他的集合中的所有Callable对象。Invoke()会返回一个Future对象的列表，通过这个列表你可以获取每一个Callable执行的结果。

一个任务可能会因为一个异常而结束，因此这时任务并不是真正意义上执行成功了。这在Future上是没有办法来判断的。

处理一个任务的容器（collection），并返回一个Future的容器。两个容器具有相同的结构，这里提交的任务容器列表和返回的Future列表存在顺序对应的关系。

下面是示例代码：

```java
ExecutorService executorService = Executors.newSingleThreadExecutor();
 
Set<Callable<String>> callables = new HashSet<Callable<String>>();
 
callables.add(new Callable<String>() {
    public String call() throws Exception {
        return "Task 1";
    }
});
callables.add(new Callable<String>() {
    public String call() throws Exception {
        return "Task 2";
    }
});
callables.add(new Callable<String>() {
    public String call() throws Exception {
        return "Task 3";
    }
});
 
List<Future<String>> futures = executorService.invokeAll(callables);
 
for(Future<String> future : futures){
    System.out.println("future.get = " + future.get());
}
 
executorService.shutdown();
```

**ExecutorService Shutdown**

当你是使用完ExecutorService后，你应该关闭它，使得线程不能持续运行。例如，你的应用程序从main()方法开始并且你的主线程退出应用程序，这时如果存在激活状态的ExecutorService，你的应用程序将仍然会保持运行。ExecutorService中激活的线程会阻止JVM关闭。

为了终止ExecutorService中的线程，你需要调用shutdown()方法。ExecutorService不会立即关闭，但是它也不会接受新的任务，直到它里面的所有线程都执行完毕，ExecutorService才会关闭。所有提交到ExecutorService中的任务会在调用shutdown()方法之前被执行。

如果你想立即关闭ExecutorService,你可以调用shutdownNow()方法。这将会尝试立即停止所有正在执行的任务，并且忽略所有提交的但未被处理的任务。对于正在执行的任务是不能确定的，也许它们停止了，也行它们执行直到结束。

### 3、ThreadPoolExecutor

Java.util.concurrent.ThreadPoolExecutor类是ExecutorSerivce接口的具体实现。ThreadPoolExecutor使用线程池中的一个线程来执行给定的任务（Runnable或者Runnable）。

ThreadPoolExecutor内部的线程池包含不定数量的线程。池中线程的数量由下面的这些变量决定：

- corePoolSize
- maximumPoolSize

当一个任务委托给线程池执行，此时如果池线程中线程数少于corePoolSize，即使池中有空闲的线程，线程池中也会创建一个新的线程。

如果任务队列是满的，corePoolSize个线程或者更多的且少于maximumPoolSize的线程正在运行，也会创建一个新的线程来执行任务。

下面图释ThreadPoolExecutor这种原理： 

![这里写图片描述](https://gitee.com/gsshy/picgo/raw/master/img/20160712174701471)

#### 创建ThreadPoolExecutor

ThreadPoolExecutor有多种构造函数。例如：

```java
int  corePoolSize  =    5;
int  maxPoolSize   =   10;
long keepAliveTime = 5000;
 
ExecutorService threadPoolExecutor =
        new ThreadPoolExecutor(
                corePoolSize,
                maxPoolSize,
                keepAliveTime,
                TimeUnit.MILLISECONDS,
                new LinkedBlockingQueue<Runnable>()
                );
```

## 三、CompletionService

`java.util.concurrent.CompletionService` 是对 `ExecutorService` 的一个功能增强封装，优化了获取异步操作结果的接口。

### 1、使用场景

假设我们要向线程池提交一批任务，并获取任务结果。一般的方式是提交任务后，从线程池得到一批 `Future` 对象集合，然后依次调用其 `get()` 方法。

这里有个问题：因为我们会要按固定的顺序来遍历 `Future` 元素，而 `get()` 方法又是阻塞的，因此如果某个 `Future` 对象执行时间太长，会使得我们的遍历过程阻塞在该元素上，无法及时从后面早已完成的 `Future` 当中取得结果。

`CompletionService` 解决了这个问题。它本身不包含线程池，创建一个 `CompletionService` 需要先创建一个 `Executor`。下面是一个例子：

```java
ExecutorService executor = Executors.newFixedThreadPool(4);
CompletionService<String> completionService = new ExecutorCompletionService<>(executor);
```

向 `CompletionService` 提交任务的方式与 `ExecutorService` 一样：

```java
completionService.submit(() -> "Hello");
```

当你需要获得结果的时候，就不同了。有了 `CompletionService`，你不需要再持有 `Future` 集合。如果要得到最早的执行结果，只需要像下面这样：

```java
String result = completionService.take().get();
```

这个 `take()` 方法返回的是最早完成的任务的结果，这个就解决了一个任务被另一个任务阻塞的问题。下面是一个完整的例子：

### 2、示例（更好的可以看代码里的测试内容）

``` java
public static void main(String[] args) throws Exception {

    ExecutorService executor;
    CompletionService<String> completionService;

    // 创建一个指定执行时长的任务的方法
    BiFunction<Integer, Integer, Callable<String>> createTask = (id, duration) -> () -> {
        log("Task " + id + " started, duration=" + duration);
        Thread.sleep(duration);
        log("Task " + id + " completed.");
        return "Result of task " + id;
    };

    ///////////////////////////////////////////////////////////////////
    System.out.println("// 示例1：像使用 ExecutorService 一样使用 CompletionService");

    // 初始化 executor 和 completionService
    executor = Executors.newFixedThreadPool(4);
    completionService = new ExecutorCompletionService<>(executor);

    // 提交任务
    List<Future<String>> results = Arrays.asList(
            completionService.submit(createTask.apply(1, 1000)),
            completionService.submit(createTask.apply(2, 800)),
            completionService.submit(createTask.apply(3, 600)),
            completionService.submit(createTask.apply(4, 400))
    );

    // 取结果
    for (Future<String> result : results) {
        log(result.get());
    }

    executor.shutdown();

    ///////////////////////////////////////////////////////////////////
    System.out.println("// 示例2：按标准方式使用 CompletionService");

    // 初始化 executor 和 completionService
    executor = Executors.newFixedThreadPool(4);
    completionService = new ExecutorCompletionService<>(executor);

    // 提交任务
    completionService.submit(createTask.apply(5, 1000));
    completionService.submit(createTask.apply(6, 800));
    completionService.submit(createTask.apply(7, 600));
    completionService.submit(createTask.apply(8, 400));

    // 取结果
    for (int i = 0; i < 4; i++) {
        log(completionService.take().get());
    }

    ///////////////////////////////////////////////////////////////////
    executor.shutdown();
}
```

这个例子的执行结果如下所示：

``` java
// 示例1：像使用 ExecutorService 一样使用 CompletionService
10:22:32:271 - Task 4 started, duration=400
10:22:32:271 - Task 3 started, duration=600
10:22:32:271 - Task 2 started, duration=800
10:22:32:271 - Task 1 started, duration=1000
10:22:32:687 - Task 4 completed.
10:22:32:888 - Task 3 completed.
10:22:33:089 - Task 2 completed.
10:22:33:303 - Task 1 completed.
10:22:33:303 - Result of task 1
10:22:33:303 - Result of task 2
10:22:33:303 - Result of task 3
10:22:33:303 - Result of task 4
// 示例2：按标准方式使用 CompletionService
10:22:33:305 - Task 5 started, duration=1000
10:22:33:305 - Task 7 started, duration=600
10:22:33:305 - Task 6 started, duration=800
10:22:33:305 - Task 8 started, duration=400
10:22:33:718 - Task 8 completed.
10:22:33:718 - Result of task 8
10:22:33:918 - Task 7 completed.
10:22:33:918 - Result of task 7
10:22:34:119 - Task 6 completed.
10:22:34:119 - Result of task 6
10:22:34:320 - Task 5 completed.
10:22:34:320 - Result of task 5
```

可以看出，在示例 1 中，虽然 Task 4 执行时间只有 400ms，但因为我们是按照 1-2-3-4 的顺序依次取结果，因此 Task 4 完成后并没有马上打印出结果来。而在示例 2 中，对每个 Task 都是在完成时立刻就将结果打印出来了。这就是 `CompletionService` 的优势所在。

### 3、原理解释

`CompletionService` 之所以能够做到这点，是因为它没有采取依次遍历 Future 的方式，而是在中间加上了一个结果队列，任务完成后马上将结果放入队列，那么从队列中取到的就是最早完成的结果。

如果队列为空，那么 `take()` 方法会阻塞直到队列中出现结果为止。此外 `CompletionService` 还提供一个 `poll()` 方法，返回值与 `take()` 方法一样，不同之处在于它不会阻塞，如果队列为空则立刻返回 null。这算是给用户多一种选择。