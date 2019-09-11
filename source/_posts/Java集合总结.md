---
title: Java集合总结
date: 2019-09-11 20:34:10 
tags: 
    - List 
    - Map 
    - Set
    - Queue 
    - Deque
---

## 概述 ##
  
Java 集合中主要包括List、Map、Set、Queue、Deque五个部分；

下面我们就一个一个的来介绍。


<!--more-->>>

## List ##

List中的元素是有序的、可重复的、主要的实现方式有动态数组和链表。

![](/image/Java集合总结/list.png)

Java中提供的实现主要有Arraylist、LinkedList、CopyOnWriteArrayList，以及古老的Vector和Stack。




## Map ##

Map是一种（key/value）的映射结构，Map中的元素是一个key只能对应一个value，不能存在重复的key。

![](/image/Java集合总结/map.png)

Java中提供Map的实现主要有HashMap、LinkedHashMap、WeakHashMap、TreeMap、ConcurrentHashMap、ConcurrentSkipListMap，另外还有两个比较古老的Map实现HashTable和Properties。


## Set ##

Set中的元素是无序、不可重复的，一般使用Map或者List实现。

![](/image/Java集合总结/set.png)

Java中提供Set的实现主要有HashSet、LinkedHashSet、TreeSet、ConcurrentSkipListSet、CopyOnWriteArraySet。




## Queue ##

Queue是一种叫做队列的数据结构，队列是遵循着FIFO原则的入队出队操作的集合，一般来说，入队是在队列尾部添加元素，出队是在队头删除元素，
但是也不一定，比如优先级队列（PriorityBlockingQueue）就有些许不同。

![](/image/Java集合总结/queue.png)


Java中提供Queue的实现主要有 ArrayBlockingQueue、PriorityBloickingQueue、LinkedBlockingQueue、SynchronousQueue、Linked
transferQueue、DelayWorkQueue、DalayQueue、ConcurrentLinkedQueue.


## Deque ##

Deque是一种特殊的队列，它的两端都可以进出元素，故而得名双端队列（Double Ended Queue）.

![](/image/Java集合总结/deque.png)

Java中提供的Deque的实现主要有ArrayDeque、LinkedBlockingDeque、ConcurrentLinkedDeque、LinkedList。