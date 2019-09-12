---
title: Java集合面试全集--Map
date: 2019-09-12 14:40:05 
tags: 
    - Java 
    - Map
    - 集合
---

## Map面试题以及解答（基于JDK1.8） ##

（1）什么是散列表？
    
     散列表也叫hash表 ，是根据hash值而进行直接进行访问的数据结构。也就是说，它通过把hash值映射到表中一个位置来访问记录，以加快查找的速度。这个映射也叫散列函数，存放记录的数组叫散列表

（2）怎么实现一个散列表？

（3）java中HashMap实现方式的演进？

（4）HashMap的容量有什么特点？

       容量为数组的长度，亦即桶的个数，默认为16，最大为2的30次方，当容量达到64时才可以树化。

（5）HashMap是怎么进行扩容的？
	 ①如果使用是默认构造方法，则第一次插入元素时初始化为默认值，容量为16，扩容门槛为12；

	 ②如果使用的是非默认构造方法，则第一次插入元素时初始化容量等于扩容门槛，扩容门槛在构造方法里等于传入容量向上最近的2的n次方；

	 ③如果旧容量大于0，则新容量等于旧容量的2倍，但不超过最大容量2的30次方，新扩容门槛为旧扩容门槛的2倍；

	 ④创建一个新容量的桶；

	 ⑤搬移元素，原链表分化成两个链表，低位链表存储在原来桶的位置，高位链表搬移到原来桶的位置加旧容量的位置；


（6） HashMap中的元素是否是有序的？
         
          不保证元素存储的顺序

（7）HashMap何时进行树化？何时进行反树化？

          树化：当容量达到64且链表的长度达到8时进行树化。
          反树化：当容量达到64且链表的长度小于6时反树化。

（8）HashMap是怎么进行缩容的？

（9）HashMap插入、删除、查询元素的时间复杂度各是多少？

    查找：时间复杂度为O(1)
    插入：时间复杂度为O(1)-O(n)
    删除:时间复杂度为O(1)-O(n)

（10）HashMap中的红黑树实现部分可以用其它数据结构代替吗？

（11）LinkedHashMap是怎么实现的？
       
        使用（数组 + 单链表 + 双向链表 + 红黑树）的存储结构

（12）LinkedHashMap是有序的吗？怎么个有序法？
  
      有序的
      如果accessOrder为false，则可以按插入元素的顺序遍历元素；
      如果accessOrder为true，则可以按访问元素的顺序遍历元素； 


（13）LinkedHashMap如何实现LRU缓存淘汰策略？
         
        默认的LinkedHashMap并不会移除旧元素，如果需要移除旧元素，则需要重写removeEldestEntry()方法设定移除策略；

	class LRU<K, V> extends LinkedHashMap<K, V> {
		
		    // 保存缓存的容量
		    private int capacity;
		
		    public LRU(int capacity, float loadFactor) {
		        super(capacity, loadFactor, true);
		        this.capacity = capacity;
		    }
		
		    /**
		    * 重写removeEldestEntry()方法设置何时移除旧元素
		    * @param eldest
		    * @return
		    */
		    @Override
		    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
		        // 当元素个数大于了缓存的容量, 就移除元素
		        return size() > this.capacity;
		    }


（14）WeakHashMap使用的数据结构？
       
       数组 + 链表

（15）WeakHashMap具有什么特性？
        
        WeakHashMap是一种弱引用map，内部的key会存储为弱引用，当jvm gc的时候，如果这些key没有强引用存在的话，会被gc回收掉，下一次当我们操作map的时候会把对应的Entry整个删除掉。
        
         WeakHashMap没有实现Clone和Serializable接口，所以不具有克隆和序列化的特性

（16）WeakHashMap通常用来做什么？
 
         WeakHashMap是一种弱引用map，内部的key会存储为弱引用，当jvm gc的时候，如果这些key没有强引用存在的话，会被gc回收掉，下一次当我们操作map的时候会把对应的Entry整个删除掉，基于这种特性，WeakHashMap特别适用于缓存处理。

（17）WeakHashMap使用String作为key是需要注意些什么？为什么？

         使用String作为key时，一定要使用new String()这样的方式声明key，才会失效，其它的基本类型的包装类型是一样的；

（18）什么是强/软/弱/虚引用？

      ①强引用：使用最普遍的引用。如果一个对象具有强引用，它绝对不会被gc回收。如果内存空间不足了，gc宁愿抛出OutOfMemoryError，也不是会回收具有强引用的对象。

	  ②软引用：如果一个对象只具有软引用，则内存空间足够时不会回收它，但内存空间不够时就会回收这部分对象。只要这个具有软引用对象没有被回收，程序就可以正常使用。

	  ③弱引用：如果一个对象只具有弱引用，则不管内存空间够不够，当gc扫描到它时就会回收它。

      ④虚引用：如果一个对象只具有虚引用，那么它就和没有任何引用一样，任何时候都可能被gc回收。

      软（弱、虚）引用必须和一个引用队列（ReferenceQueue）一起使用，当gc回收这个软（弱、虚）引用引用的对象时，会把这个软（弱、虚）引用放到这个引用队列中。

（19）红黑树具有哪些特性？

	 	①每个节点或者是黑色，或者是红色。
		②根节点是黑色。
		③每个叶子节点（NIL）是黑色。 [注意：这里叶子节点，是指为空(NIL或NULL)的叶子节点！]
		④如果一个节点是红色的，则它的子节点必须是黑色的。
		⑤从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。[这里指到叶子节点的路径]

（20）TreeMap就有序的吗？怎么个有序法？
     	
		按key的大小排序有两种方式，一种是key实现Comparable接口，一种方式通过构造方法传入比较器。

（21）TreeMap是否需要扩容？

（22）什么是左旋？什么是右旋？

（23）红黑树怎么插入元素？

（24）红黑树怎么删除元素？

（25）为什么要进行平衡？

（26）如何实现红黑树的遍历？

（27）TreeMap中是怎么遍历的？

（28）TreeMap插入、删除、查询元素的时间复杂度各是多少？

（29）HashMap在多线程环境中什么时候会出现问题？

（30）ConcurrentHashMap的存储结构？

（31）ConcurrentHashMap是怎么保证并发安全的？

（32）ConcurrentHashMap是怎么扩容的？

（33）ConcurrentHashMap的size()方法的实现知多少？

（34）ConcurrentHashMap是强一致性的吗？

（35）ConcurrentHashMap不能解决什么问题？

（36）ConcurrentHashMap中哪些地方运用到分段锁的思想？

（37）什么是伪共享？怎么避免伪共享？

（38）什么是跳表？

（40）ConcurrentSkipList是有序的吗？

（41）ConcurrentSkipList是如何保证线程安全的？

（42）ConcurrentSkipList插入、删除、查询元素的时间复杂度各是多少？

（43）ConcurrentSkipList的索引具有什么特性？

（44）为什么Redis选择使用跳表而不是红黑树来实现有序集合？

