---
title: Java多线程（二）之线程的创建和使用 
date: 2019-08-09 20:45:57 
tags: Java,多线程
---

1.线程的创建和启动

 Java语言的JVM允许程序运行多个线程，它通过java.lang.Thread类来体现。

 Thread类的特性
   
  --->每个线程都是通过某个特定Thread对象的run()方法来完成操作的，经常把run()方法的主体称为线程体

  --->通过该Thread对象的start()方法来启动这个线程，而非直接调用run()
<!--more-->
2.Thread类

  --->构造器

   --->  Thread()：创建新的Thread对象

   --->  Thread(String threadname)：创建线程并指定线程实例名

   --->  Thread(Runnable target)：指定创建线程的目标对象，它实现了Runnable接口中的run方法

   --->  Thread(Runnable target, String name)：创建新的Thread对象
 
      
3.API中创建线程的四种方式
    
 --->JDK1.5之前创建新执行线程有两种方法：

   --->继承Thread类的方式

   --->实现Runnable接口的方式

 --->JDK5.0新增线程创建方式的两种方法：

 --->  实现Callable接口的方式

 --->  利用线程池的方式（企业中一般使用这种方式）


方式一：继承Thread类

   1) 定义子类继承Thread类。

   2) 子类中重写Thread类中的run方法。

   3) 创建Thread子类对象，即创建了线程对象。

   4) 调用线程对象start方法：启动线程，调用run方法。

   注意点：
 
   1.如果自己手动调用run()方法，那么就只是普通方法，没有启动多线程模式。

   2. run()方法由JVM调用，什么时候调用，执行的过程控制都有操作系统的CPU调度决定。

   3. 想要启动多线程，必须调用start方法。

   4. 一个线程对象只能调用一次start()方法启动，如果重复调用了，则将抛出以上的异常“IllegalThreadStateException”。
   


方式二：实现Runnable接口

  1) 定义子类，实现Runnable接口。

  2) 子类中重写Runnable接口中的run方法。

  3) 通过Thread类含参构造器创建线程对象。

  4) 将Runnable接口的子类对象作为实际参数传递给Thread类的构造器中。

  5) 调用Thread类的start方法：开启线程，调用Runnable子类接口的run方法。




方式三：实现Callable接口（和Runnable接口方式很相似，但也有不同后面会有对比）

   Future接口

  1）可以对具体Runnable、Callable任务的执行结果进行取消、查询是否完成、获取结果等。

  2） FutrueTask是Futrue接口的唯一的实现类

  3）FutureTask 同时实现了Runnable, Future接口。它既可以作为

  4）Runnable被线程执行，又可以作为Future得到Callable的返回值

方式四：使用线程池
  
 
   背景：经常创建和销毁、使用量特别大的资源，比如并发情况下的线程，对性能影响很大。

   思路：提前创建好多个线程，放入线程池中，使用时直接获取，使用完放回池中。可以避免频繁创建销毁、实现重复利用。

 好处：

   提高响应速度（减少了创建新线程的时间）

   降低资源消耗（重复利用线程池中线程，不需要每次都创建）

   便于线程管理

   --->corePoolSize：核心池的大小

   --->maximumPoolSize：最大线程数

   --->keepAliveTime：线程没有任务时最多保持多长时间后会终止


线程池相关API


  JDK 5.0起提供了线程池相关API：ExecutorService 和 Executors

ExecutorService：真正的线程池接口。常见子类ThreadPoolExecutor

   --->  void execute(Runnable command) ：执行任务/命令，没有返回值，一般用来执行Runnable

   --->  <T> Future<T> submit(Callable<T> task)：执行任务，有返回值，一般又来执行Callable

   --->  void shutdown() ：关闭连接池



Executors：工具类、线程池的工厂类，用于创建并返回不同类型的线程池 

   --->  Executors.newCachedThreadPool()：创建一个可根据需要创建新线程的线程池

   --->  Executors.newFixedThreadPool(n); 创建一个可重用固定线程数的线程池

   --->  Executors.newSingleThreadExecutor() ：创建一个只有一个线程的线程池

   --->  Executors.newScheduledThreadPool(n)：创建一个线程池，它可安排在给定延迟后运行命令或者定期地执行。
  

4.继承方式和实现方式的联系与区别 
   
  public class Thread extends Object implements Runnable

  区别:

  --->  继承Thread：线程代码存放Thread子类run方法中。

  --->  实现Runnable：线程代码存在接口的子类的run方法。

实现方式的好处:

  --->  避免了单继承的局限性

  --->  多个线程可以共享同一个接口实现类的对象，非常适合多个相同线程来处理同一份资源。

联系：

  --->  当你去看thread源码的时候你会发现，Thread 实现了Runnable 接口


5.Callable 和Runnable的区别与联系
  
  区别：与使用Runnable相比， Callable功能更强大些

   --->  相比run()方法，可以有返回值

   --->  方法可以抛出异常

   --->  支持泛型的返回值

   --->  需要借助FutureTask类，比如获取返回结果



6.Thread类的有关方法

  --->   void start(): 启动线程，并执行对象的run()方法

  --->   run(): 线程在被调度时执行的操作

  --->   String getName(): 返回线程的名称

  --->   void setName(String name):设置该线程名称

  --->   static Thread currentThread(): 返回当前线程。在Thread子类中就是this，通常用于主线程和Runnable实现类

  --->   static void yield()：线程让步

   --->      暂停当前正在执行的线程，把执行机会让给优先级相同或更高的线程
   --->      若队列中没有同优先级的线程，忽略此方法

  --->   join() ：当某个程序执行流中调用其他线程的 join() 方法时，调用线程将被阻塞，直到 join() 方法加入的 join 线程执行完为止

   --->   低优先级的线程也可以获得执行

  --->   static void sleep(long millis)：(指定时间:毫秒)

   --->   令当前活动线程在指定时间段内放弃对CPU控制,使其他线程有机会被执行,时间到后重排队。

   --->   抛出InterruptedException异常

  --->   stop(): 强制线程生命期结束，不推荐使用

  --->   boolean isAlive()：返回boolean，判断线程是否还活着


7.线程的分类

  Java中的线程分为两类：一种是守护线程，一种是用户线程。

   --->  它们在几乎每个方面都是相同的，唯一的区别是判断JVM何时离开。

   --->  守护线程是用来服务用户线程的，通过在start()方法前调用thread.setDaemon(true)可以把一个用户线程变成一个守护线程。

   --->  Java垃圾回收就是一个典型的守护线程。

   --->  若JVM中都是守护线程，当前JVM将退出。

   --->  形象理解：兔死狗烹，鸟尽弓藏
