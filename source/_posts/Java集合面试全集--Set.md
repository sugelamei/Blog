---
title: Java集合面试全集--Set
date: 2019-09-12 08:45:31 
tags: 
    - Java 
    - Set
    - 集合
---

## Set面试题以及解答（基于JDK1.8） ##

（1）HashSet怎么保证添加元素不重复？

       HashSet是Set的一种实现方式，底层主要使用HashMap来确保元素不重复。 
       通过hashCode以及equal保证元素不重复
        HashSet内部使用HashMap的key存储元素，以此来保证元素不重复；

（2）HashSet是有序的吗？

      HashSet是无序的，因为HashMap的key是无序的；

（3）HashSet是否允许null元素？

        HashSet中允许有一个null元素，因为HashMap允许key为null；
       

（4）Set是否有get()方法？
   
       HashSet是没有get()方法的；

（5）LinkedHashSet是有序的吗？怎么个有序法？
         
         LinkedHashSet是有序的，它是按照插入的顺序排序的

（6）LinkedHashSet支持按元素访问顺序排序吗？

        LinkedHashSet是不支持按访问顺序对元素排序的，只能按插入顺序排序。

（8）TreeSet真的是使用TreeMap来存储元素的吗？
     
      我们知道TreeSet里面实际上是使用的NavigableMap来存储元素，虽然大部分时候这个map确实是TreeMap，但不是所有时候都是TreeMap。
       TreeSet的底层不完全是使用TreeMap来实现的，更准确地说，应该是NavigableMap。

（9）TreeSet是有序的吗？怎么个有序法？
            
      TreeSet是有序的，TreeSet实现了NavigableMap接口，它的有序性主要依赖于NavigableMap的有序性，而NavigableMap又继承自SortedMap，这个接口的有序性是指按照key的自然排序保证的有序性，而key的自然排序又有两种实现方式，一种是key实现Comparable接口，一种是构造方法传入Comparator比较器。

（10）TreeSet和LinkedHashSet有何不同？

          排序方式不同；
          
		LinkedHashSet并没有实现SortedSet接口，它的有序性主要依赖于LinkedHashMap的有序性，所以它的有序性是指按照插入顺序保证的有序性；
		
		而TreeSet实现了NavigableMap接口，它的有序性主要依赖于NavigableMap的有序性，而NavigableMap又继承自SortedMap，这个接口的有序性是指按照key的自然排序保证的有序性，而key的自然排序又有两种实现方式，一种是key实现Comparable接口，一种是构造方法传入Comparator比较器。

（11）TreeSet和SortedSet有什么区别和联系？

     TreeSet实现了NavigableMap接口，NavigableMap又继承自SortedMap；
         

（12）CopyOnWriteArraySet是用Map实现的吗？

    CopyOnWriteArraySet底层是使用CopyOnWriteArrayList存储元素的，所以它并不是使用Map来存储元素的。

（13）CopyOnWriteArraySet是有序的吗？怎么个有序法？
 
      CopyOnWriteArraySet是有序的，因为底层其实是数组，按照插入顺序排序

（14）CopyOnWriteArraySet怎么保证并发安全？

        CopyOnWriteArraySet是并发安全的，而且实现了读写分离

（15）CopyOnWriteArraySet以何种方式保证元素不重复？
 
    CopyOnWriteArraySet通过调用CopyOnWriteArrayList的addIfAbsent()方法来保证元素不重复；
        

（16）如何比较两个Set中的元素是否完全一致？

         Set中的元素并不重复，所以只要先比较两个Set的元素个数是否相等，再作一次两层循环就可以

     private static <T> boolean eq(Set<T> set1, Set<T> set2) {
        if (set1.size() != set2.size()) {
            return false;
        }

        for (T t : set1) {
            // contains相当于一层for循环
            if (!set2.contains(t)) {
                return false;
            }
        }

        return true;
    }


（17）ConcurrentSkipListSet的底层是ConcurrentSkipListMap吗？

      ConcurrentSkipListSet底层是通过ConcurrentNavigableMap来实现的

（18）ConcurrentSkipListSet是有序的吗？怎么个有序法？

         ConcurrentSkipListSet有序的，基于元素的自然排序或者通过比较器确定的顺序

 

 (19)Set大汇总

![](/image/Java集合总结/set总结.png)


从中我们可以发现一些规律：

①除了HashSet其它Set都是有序的；

②实现了NavigableSet或者SortedSet接口的都是自然顺序的；

③使用并发安全的集合实现的Set也是并发安全的；

④TreeSet虽然不是全部都是使用的TreeMap实现的，但其实都是跟TreeMap相关的（TreeMap的子Map中组合了TreeMap）；

⑤ConcurrentSkipListSet虽然不是全部都是使用的ConcurrentSkipListMap实现的，但其实都是跟ConcurrentSkipListMap相关的（ConcurrentSkipListeMap的子Map中组合了ConcurrentSkipListMap）；
              

      