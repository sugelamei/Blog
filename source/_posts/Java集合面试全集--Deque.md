---
title: Java集合面试全集--Deque
date: 2019-09-16 21:59:55  
tags: 
    - Java 
    - Deque
    - 集合
---

## Deque面试题以及解答（基于JDK1.8） ##

（1）什么是双端队列？
    
    双端队列是一种特殊的队列，它的两端都可以进出元素，故而得名双端队列；

（2）ArrayDeque是怎么实现双端队列的？
       
      ArrayDeque是一种以数组方式实现的双端队列


（3）ArrayDeque是有界的吗？

      不是，ArrayDeque容量不足时是会扩容的，每次扩容容量增加一倍
<!--more-->>>
（4）LinkedList与ArrayDeque的对比？
      
     ①前者使用链表实现，后者使用数组实现
     ②二者都是非线程安全的，都实现了Deque
     

（5）双端队列是否可以作为栈使用？
      
      ArrayDeque可以直接作为栈使用，入栈出栈只要都操作队列头就可以了

（6）LinkedBlockingDeque是怎么实现双端队列的？

          使用链表实现双端队列

（7）LinkedBlockingDeque是怎么保证并发安全的？
 
     使用重入锁ReentrantLock + Condition保证并发安全的
         

（8）ConcurrentLinkedDeque是怎么实现双端队列的？

        使用链表实现双端队列

（9）ConcurrentLinkedDeque是怎么保证并发安全的？

        CAS + 自旋

（10）LinkedList是List和Deque的集合体？

       LinkedList实现了List和Deque，具有list和Deque的特性；
 