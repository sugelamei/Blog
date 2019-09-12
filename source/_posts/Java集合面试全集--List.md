---
title: Java集合面试全集--List
date: 2019-09-12 08:45:31 
tags: 
    - Java 
    - List
    - 集合
---

## List面试题以及解答（基于JDK1.8） ##

（1）ArrayList和LinkedList有什么区别？
     
 		
        ①ArrayList是基于动态数组实现的数据结构，LinkedList是基于双向链表的数据结构。
        ②对于随机访问get，ArrayList优于LinkedList，因为LinkedList需要移动指针。
      	③对于新增删除add和remove，LinkedList优于ArrayList，因为ArrayList需要移动数据。

（2）ArrayList是怎么扩容的？

        ①调用无参构造器创建一个空的object数组
        ②当调用add方法时，调用ensureCapacityInternal(size + 1)确定内部容量
		③调用ensureExplicitCapacity(calculateCapacity(elementData, minCapacity))方法确保扩展能力，其中calculateCapacity(elementData, minCapacity)是计算当前需要的容量大小
		④使用grow( minCapacity)来实现容量扩展为原来的1.5倍（oldCapacity + (oldCapacity >> 1)）	
        ⑤使用Arrays.copyOf(elementData, newCapacity)将老数组的元素拷贝到新数组中。
		
		
（3）ArrayList插入、删除、查询元素的时间复杂度各是多少？
    
        ①get(int index) 时间复杂度为O(1)
		②add(E e) 时间复杂度为O(1) 直接在最后添加
        ③add(int index, E element) 时间复杂度为O(n) 需要移动数据
		④remove(int index) 时间复杂度为O(n) 需要移动数据 
        ⑤remove(Object o) 时间复杂度为O(n) 需要遍历集合找到这个元素
     
       

（4）怎么求两个集合的并集、交集、差集？
        
         ①交集  retainAll(Collection<?> c)  
         ②并集  addAll(Collection<? extends E> c)
         ③差集  removeAll(Collection<?> c)
         ④无重复并集 removeAll(Collection<?> c) + addAll(Collection<? extends E> c)

（5）ArrayList是怎么实现序列化和反序列化的？
         
          ①使用writeObject(java.io.ObjectOutputStream s)方法进行序列化
		  ②使用readObject(java.io.ObjectInputStream s)方法进行反序列化
          ③transient Object[] elementData 防止整个数组反序列化
          ④序列化和反序列只对数组中的元素进行，而不是整个数组
    


（6）集合的方法toArray()有什么问题？
        
       ①使用toArray()时强转会报ClassCastException
          如： String[] strings = (String[]) list1.toArray();

       ②此时应该使用T[] toArray(T[] a)方法
          如：String[] strings = list1.toArray(new String[list1.size()]);

（7）什么是fail-fast和 fail—safe？

        快速失败（fail—fast）：在用迭代器遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了修改（增加、删除、修改），则会抛出Concurrent Modification Exception。

        原理：迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个 modCount 变量。集合在被遍历期间如果内容发生变化，就会改变modCount的值。每当迭代器使用hashNext()/next()遍历下一个元素之前，都会检测modCount变量是否为expectedmodCount值，是的话就返回遍历；否则抛出异常，终止遍历。

        注意：这里异常的抛出条件是检测到 modCount！=expectedmodCount 这个条件。如果集合发生变化时修改modCount值刚好又设置为了expectedmodCount值，则异常不会抛出。因此，不能依赖于这个异常是否抛出而进行并发操作的编程，这个异常只建议用于检测并发修改的bug。

        场景：java.util包下的集合类都是快速失败的，不能在多线程下发生并发修改（迭代过程中被修改）。


        安全失败（fail—safe）：采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。
          
        原理：由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发Concurrent Modification Exception。

		缺点：基于拷贝内容的优点是避免了Concurrent Modification Exception，但同样地，迭代器并不能访问到修改后的内容，即：迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改迭代器是不知道的。

        场景：java.util.concurrent包下的容器都是安全失败，可以在多线程下并发使用，并发修改。


（8）LinkedList是单链表还是双链表实现的？

         双向链表：

        private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }

（9）LinkedList除了作为List还有什么用处？

         ①LinkedList还是一个双端队列，具有队列、双端队列、栈的特性

         ②LinkedList在功能上等于ArrayList + ArrayDeque


（10）LinkedList插入、删除、查询元素的时间复杂度各是多少？

		①get(int index) 时间复杂度为O(n) 依次遍历
		②add(E e) 时间复杂度为O(1) 直接在最后添加
        ③add(int index, E element) 时间复杂度为O(1) 先查找元素位置，然后
		④remove(int index) 时间复杂度为O(n) 先查找元素位置，然后进行指针操作
        ⑤remove(Object o) 时间复杂度为O(n) 需要遍历集合找到这个元素


（11）什么是随机访问和顺序访问？
      
      随机访问：支持任意位置元素的读取
      
      顺序访问：支持从第一个元素开始的顺序读取。

（12）哪些集合支持随机访问？他们都有哪些共性？

      支持随机访问的集合：ArrayList、CopyOnWriteArrayList、Vector、Stack

      共性：底层实现都是由数组实现的，都实现了RandomAccess接口

（13）CopyOnWriteArrayList是怎么保证并发安全的？

      CopyOnWriteArrayList的写操作都要先拷贝一份新数组，在新数组中做修改，修改完了再用新数组替换老数组

（14）CopyOnWriteArrayList的实现采用了什么思想？

      CopyOnWriteArrayList采用读写分离的思想，读操作不加锁，写操作加锁，且写操作占用较大内存空间，所以适用于读多写少的场合

（15）CopyOnWriteArrayList是不是强一致性的？
      
      CopyOnWriteArrayList只保证最终一致性，不保证实时一致性

（16）CopyOnWriteArrayList适用于什么样的场景？
      
       适用于读多写少的场合

（17）CopyOnWriteArrayList插入、删除、查询元素的时间复杂度各是多少？
       
        ①get(int index) 时间复杂度为O(1) 依次遍历
		②add(E e) 时间复杂度为O(1) 直接在最后添加
        ③add(int index, E element) 时间复杂度为O(1) 
		④remove(int index) 时间复杂度为O(n) 先查找元素位置，然后进行指针操作
        ⑤remove(Object o) 时间复杂度为O(n) 需要遍历集合找到这个元素


（18）CopyOnWriteArrayList为什么没有size属性？
      
         数组的长度就是集合的大小

（19）比较古老的集合Vector和Stack有什么缺陷？
          
       Vector 是线程安全的动态数组，Vector 与 ArrayList 基本是一致的，唯一不同的是 Vector 是线程安全的，会在可能出现线程安全的方法前面加上 synchronized 关键字导致性能低下；

        Stack 是继承自 Vector 基于动态数组实现的线程安全栈，其实现也借助了Vector的数据结构和方法。简单的说就是一个push操作：往数组中最后添加元素，pop操作：取出最后一个元素，达到先入后出的原则