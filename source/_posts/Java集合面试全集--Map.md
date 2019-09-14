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

       
 https://www.cnblogs.com/absfree/p/5508570.html

（3）java中HashMap实现方式？
 
     数组 +链表 +红黑树

（4）HashMap的容量有什么特点？

       容量为数组的长度，亦即桶的个数，默认为16，最大为2的30次方，当容量达到64时才可以树化。

<!--more-->>>

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

      使用HashMap时还需要注意一点，它不会动态地进行缩容，也就是说，你不应该保留一个已经删除过大量Entry的HashMap（如果不打算继续添加元素的话），此时它的buckets数组经过多次扩容已经变得非常大了，这会占用非常多的无用内存，这样做的好处是不用多次对数组进行扩容或缩容操作。不过一般也不会出现这种情况，如果遇见了，请毫不犹豫地丢掉它，或者把数据转移到一个新的HashMap。


（9）HashMap插入、删除、查询元素的时间复杂度各是多少？

    查找：时间复杂度为O(1)
    插入：时间复杂度为O(1)-O(n)
    删除:时间复杂度为O(1)-O(n)

（10）HashMap中的红黑树实现部分可以用其它数据结构代替吗？

       不能，因为在这个情况下，只有红黑书在极端的情况下能保持极好的性能，将时间复杂度保持在O(logN)。

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
     	
	按key的大小排序有两种方式，一种是key实现Comparable接口，一种方式通过构造方法传入比较器comparator。

（21）TreeMap是否需要扩容？

      不需要扩容

（22）什么是左旋？什么是右旋？
        
       左旋，就是以某个节点为支点向左旋转。
 
       右旋，就是以某个节点为支点向右旋转。
       
       具体如何旋转 可以参考[https://www.cnblogs.com/yangecnu/p/Introduce-Red-Black-Tree.html](https://www.cnblogs.com/yangecnu/p/Introduce-Red-Black-Tree.html)


（23）红黑树怎么插入元素？

      具体查看以下链接：
      
       https://blog.csdn.net/goodluckwhh/article/details/11804733

（24）红黑树怎么删除元素？
   
      具体查看以下链接：

       https://blog.csdn.net/goodluckwhh/article/details/12718233

（25）为什么要进行平衡？
  
       平衡是为了保证查询、删除、查询的时间复杂度问题，为了提高查询、删除、查询的速度
        

（26）如何实现红黑树的遍历？

        ①前序遍历，先遍历我，再遍历我的左子节点，最后遍历我的右子节点；

		②中序遍历，先遍历我的左子节点，再遍历我，最后遍历我的右子节点；
		
		③后序遍历，先遍历我的左子节点，再遍历我的右子节点，最后遍历我；

（27）TreeMap中是怎么遍历的？

        @Override
		public void forEach(BiConsumer<? super K, ? super V> action) {
		    Objects.requireNonNull(action);
		    // 遍历前的修改次数
		    int expectedModCount = modCount;
		    // 执行遍历，先获取第一个元素的位置，再循环遍历后继节点
		    for (Entry<K, V> e = getFirstEntry(); e != null; e = successor(e)) {
		        // 执行动作
		        action.accept(e.key, e.value);
		
		        // 如果发现修改次数变了，则抛出异常
		        if (expectedModCount != modCount) {
		            throw new ConcurrentModificationException();
		        }
		    }
		}



  ① 寻找第一个节点，从根节点开始找最左边的节点，即最小的元素。

		final Entry<K,V> getFirstEntry() {
		        Entry<K,V> p = root;
		        // 从根节点开始找最左边的节点，即最小的元素
		        if (p != null)
		            while (p.left != null)
		                p = p.left;
		        return p;
		    }

      


②循环遍历后继节点

		static <K,V> TreeMap.Entry<K,V> successor(Entry<K,V> t) {
		    if (t == null)
		        // 如果当前节点为空，返回空
		        return null;
		    else if (t.right != null) {
		        // 如果当前节点有右子树，取右子树中最小的节点
		        Entry<K,V> p = t.right;
		        while (p.left != null)
		            p = p.left;
		        return p;
		    } else {
		        // 如果当前节点没有右子树
		        // 如果当前节点是父节点的左子节点，直接返回父节点
		        // 如果当前节点是父节点的右子节点，一直往上找，直到找到一个祖先节点是其父节点的左子节点为止，返回这个祖先节点的父节点
		        Entry<K,V> p = t.parent;
		        Entry<K,V> ch = t;
		        while (p != null && ch == p.right) {
		            ch = p;
		            p = p.parent;
		        }
		        return p;
		    }
		}


（28）TreeMap插入、删除、查询元素的时间复杂度各是多少？

          插入、删除、查询的时间的复杂度均为O(lgN)
         

（29）HashMap在多线程环境中什么时候会出现问题？
         
     会出现线程安全问题 

（30）ConcurrentHashMap的存储结构？
   
     数组+ 链表 + 红黑书

（31）ConcurrentHashMap是怎么保证并发安全的？
         
       自旋 +CAS +Synchronized + 分段锁  

（32）ConcurrentHashMap是怎么扩容的？

       采用多线程扩容。整个扩容过程，通过CAS设置sizeCtl，transferIndex等变量协调多个线程进行并发扩容。
       多线程无锁扩容的关键就是通过CAS设置sizeCtl与transferIndex变量，协调多个线程对table数组中的node进行迁移。

         

（33）ConcurrentHashMap的size()方法的实现知多少？
        
     获取元素个数是把所有的段（包括baseCount和CounterCell）相加起来得到的
        
      

（34）ConcurrentHashMap是强一致性的吗？
        
      查询操作是不会加锁的，所以ConcurrentHashMap不是强一致性的

（35）ConcurrentHashMap不能解决什么问题？

		       private static final Map<Integer, Integer> map = new ConcurrentHashMap<>();
		
		public void unsafeUpdate(Integer key, Integer value) {
		    Integer oldValue = map.get(key);
		    if (oldValue == null) {
		        map.put(key, value);
		    }
		}

       多线程同时访问时存在问题值被覆盖的问题

（36）ConcurrentHashMap中哪些地方运用到分段锁的思想？
    
         插入（put）  删除（remove）  

（37）什么是伪共享？怎么避免伪共享？

     伪共享：计算机系统中为了解决主内存与CPU运行速度的差距，在CPU与主内存之间添加了一级或者多级高速缓冲存储器（Cache），这个Cache一般是集成到CPU内部的，所以也叫 CPU Cache。
  
     避免伪共享的方式：

     JDK8之前一般都是通过字节填充的方式来避免，也就是创建一个变量的时候使用填充字段填充该变量所在的缓存行，这样就避免了多个变量存在同一个缓存行

       在JDK8中提供了一个sun.misc.Contended注解，用来解决伪共享问题

（38）什么是跳表？

      跳表是一个随机化的数据结构，实质就是一种可以进行二分查找的有序链表。
      
      跳表在原有的有序链表上面增加了多级索引，通过索引来实现快速查找。

      跳表不仅能提高搜索性能，同时也可以提高插入和删除操作的性能

（40）ConcurrentSkipListMap是有序的吗？

        是有序的


（41）ConcurrentSkipListMap是如何保证线程安全的？

            它利用了volatile关键字、Unsafe的CAS方法、自旋锁等并发技巧实现了无锁算法


（42）ConcurrentSkipListMap插入、删除、查询元素的时间复杂度各是多少？

      跳表查询、插入、删除的时间复杂度为O(lgN)

（43）ConcurrentSkipListMap的索引具有什么特性？

          每个节点按照一定的概率生成了多级索引，下层的索引必定包含上层索引

（44）为什么Redis选择使用跳表而不是红黑树来实现有序集合？

		在跳表中，要查找区间的元素，我们只要定位到两个区间端点在最低层级的位置，然后按顺序遍历元素就可以了，非常高效。
		
		而红黑树只能定位到端点后，再从首位置开始每次都要查找后继节点，相对来说是比较耗时的。
		
		此外，跳表实现起来很容易且易读，红黑树实现起来相对困难，所以Redis选择使用跳表来实现有序集合。

