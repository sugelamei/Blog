---
title: Java集合面试全集--Queue
date: 2019-09-15 21:39:36 
tags: 
    - Java 
    - Queue
    - 集合
---


## Queue面试题以及解答（基于JDK1.8） ##

（1）什么是堆？什么是堆化？

		堆是一种特殊的树，只要满足下面两个条件，它就是一个堆：
		
		①堆是一颗完全二叉树；
		
		②堆中某个节点的值总是不大于（或不小于）其父节点的值。
		
		其中，我们把根节点最大的堆叫做大顶堆，根节点最小的堆叫做小顶堆。

      堆化：把一个无序整数数组变成一个堆数组。如果是最小堆，每个元素A[i]，我们将得到A[i * 2 + 1] >= A[i]和A[i * 2 + 2] >= A[i]

（2）什么是优先级队列？

    优先级队列，是0个或多个元素的集合，集合中的每个元素都有一个权重值，每次出队都弹出优先级最大或最小的元素

<!--more-->>>

（3）PriorityQueue是怎么实现的？
     
        PriorityQueue是一个小顶堆，底层使用数组，入队就是堆的插入元素的实现，出队就是堆的删除元素的实现
    

（4）PriorityQueue是有序的吗？

    PriorityQueue不是有序的，只有堆顶存储着最小的元素

（5）PriorityQueue入队、出队的时间复杂度各是多少？

           入队的时间复杂度O（lgN）
           出队的时间复杂度O（lgN）

（6）PriorityQueue是否需要扩容？扩容规则呢？

    PriorityQueue需要扩容

    扩容规则：

        ①当数组比较小（小于64）的时候每次扩容容量翻倍；

        ②当数组比较大的时候每次扩容只增加一半的容量；

（7）ArrayBlockingQueue的实现方式？
 
        ArrayBlockingQueue是java并发包下一个以数组实现的阻塞队列


（8）ArrayBlockingQueue是否需要扩容？
       
         ArrayBlockingQueue不需要扩容，因为是初始化时指定容量，并循环利用数组

（9）ArrayBlockingQueue怎么保证线程安全？

        利用重入锁和两个条件（空和满）保证并发安全

（9）ArrayBlockingQueue有什么缺点？
            
    ①队列长度固定且必须在初始化时指定，所以使用之前一定要慎重考虑好容量；

	②如果消费速度跟不上入队速度，则会导致提供者线程一直阻塞，且越阻塞越多，非常危险；
		
	③只使用了一个锁来控制入队出队，效率较低

（10）LinkedBlockingQueue的实现方式？

       LinkedBlockingQueue是java并发包下一个以单链表实现的阻塞队列

（11）LinkedBlockingQueue是有界的还是无界的队列？
    
      LinkedBlockingQueue是有界队列，不传入容量时默认为最大int值；

（12）LinkedBlockingQueue怎么保证线程安全？

       ReentrantLock + Condition  LinkedBlockingQueue采用两把锁的锁分离技术实现入队出队互不阻塞

（13）LinkedBlockingQueue与ArrayBlockingQueue对比？
 
       ①后者入队出队采用一把锁，导致入队出队相互阻塞，效率低下；

	   ②前者入队出队采用两把锁，入队出队互不干扰，效率较高；
		
	   ③二者都是有界队列，如果长度相等且出队速度跟不上入队速度，都会导致大量线程阻塞；
		
	   ④前者如果初始化不传入初始容量，则使用最大int值，如果出队速度跟不上入队速度，会导致队列特别长，占用大量内存；
       
       ⑤前者使用链表，后者使用数组；
 

（14）SynchronousQueue的实现方式？
     
       SynchronousQueue有两种实现方式，一种是公平（队列）方式，一种是非公平（栈）方式

（15）SynchronousQueue真的是无缓冲的吗？
 
       通过源码分析，我们可以发现其实SynchronousQueue内部或者使用栈或者使用队列来存储包含线程和元素值的节点，如果同一个模式的节点过多的话，它们都会存储进来，且都会阻塞着，所以，严格上来说，SynchronousQueue并不能算是一个无缓冲队列。

（16）SynchronousQueue怎么保证线程安全？
      
        
     volatile + ReentranLock + CAS + LockSupport

（17）SynchronousQueue的公平模式和非公平模式有什么区别？
           公平模式使用数据结构为队列，非公平魔兽使用数据结构为栈
            

（18）SynchronousQueue在高并发情景下会有什么问题？
     
           如果生产、消费的速度相差很大的情况下会导致系统中过多的线程处于阻塞状态。

（19）PriorityBlockingQueue的实现方式？

      数组

（20）PriorityBlockingQueue是否需要扩容？

       PriorityBlockingQueue扩容时旧容量小于64则翻倍，旧容量大于64则增加一半
       PriorityBlockingQueue扩容时使用一个单独变量的CAS操作来控制只有一个线程进行扩容 

（21）PriorityBlockingQueue怎么保证线程安全？

     PriorityBlockingQueue使用一个锁+一个notEmpty条件控制并发安全

（22）PriorityBlockingQueue为什么不需要notFull条件？

       因为PriorityBlockingQueue在入队的时候如果没有空间了是会自动扩容的，也就不存在队列满了的状态，也就是不需要等待通知队列不满了可以放元素了，所以也就不需要notFull条件了。

（23）什么是双重队列？

    放取元素使用同一个队列，队列中的节点具有两种模式，一种是数据节点，一种是非数据节点。

	放元素时先跟队列头节点对比，如果头节点是非数据节点，就让他们匹配，如果头节点是数据节点，就生成一个数据节点放在队列尾端（入队）。

	取元素时也是先跟队列头节点对比，如果头节点是数据节点，就让他们匹配，如果头节点是非数据节点，就生成一个非数据节点放在队列尾端（入队）。

（24）LinkedTransferQueue是怎么实现阻塞队列的？

            调用LockSupport.park()或LockSupport.parkNanos阻塞

（25）LinkedTransferQueue是怎么控制并发安全的？

      LinkedTransferQueue全程都没有使用synchronized、重入锁等比较重的锁，基本是通过 自旋+CAS+LockSupport 实现 

（26）LinkedTransferQueue与SynchronousQueue有什么异同？
      ①在java8中两者的实现方式基本一致，都是使用的双重队列；

	  ②前者完全实现了后者，但比后者更灵活；
			
	  ③后者不管放元素还是取元素，如果没有可匹配的元素，所在的线程都会阻塞；
			
	  ④前者可以自己控制放元素是否需要阻塞线程，比如使用四个添加元素的方法就不会阻塞线程，只入队元素，使用transfer()会阻塞线程；
			
	  ⑤取元素两者基本一样，都会阻塞等待有新的元素进入被匹配到；
        

（27）ConcurrentLinkedQueue是阻塞队列吗？

        ConcurrentLinkedQueue只实现了Queue接口，并没有实现BlockingQueue接口，所以它不是阻塞队列，也不能用于线程池中，但是它是线程安全的，可用于多线程环境中。

（28）ConcurrentLinkedQueue如何保证并发安全？

        ConcurrentLinkedQueue使用（CAS+自旋）更新头尾节点控制出队入队操作；

（29）ConcurrentLinkedQueue能用于线程池吗？

        ConcurrentLinkedQueue不能用在线程池中；


（30）ConcurrentLinkedQueue与LinkedBlockingQueue对比？

       ①两者都是线程安全的队列；

	   ②两者都可以实现取元素时队列为空直接返回null，后者的poll()方法可以实现此功能；
		
	   ③前者全程无锁，后者全部都是使用重入锁控制的；
		
	   ④前者效率较高，后者效率较低；
		
	   ⑤前者无法实现如果队列为空等待元素到来的操作；
		
       ⑥前者是非阻塞队列，后者是阻塞队列；
		
	   ⑦前者无法用在线程池中，后者可以；

（31）DelayQueue是阻塞队列吗？

       从继承体系可以看到，DelayQueue实现了BlockingQueue，所以它是一个阻塞队列。

（32）DelayQueue的实现方式？

       从属性我们可以知道，延时队列主要使用优先级队列来实现

（33）DelayQueue主要用于什么场景？

       DelayQueue是java并发包下的延时阻塞队列，常用于实现定时任务。


（34）DelayQueue如何保证并发安全？
   
      DelayQueue使用重入锁和条件来控制并发安全