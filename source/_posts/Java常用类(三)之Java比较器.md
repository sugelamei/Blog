---
title: Java常用类(三)之Java比较器
date: 2019-08-12 16:49:47 
tags: 
    - Java 
    - 常用类
---

在Java中经常会涉及到对象数组的排序问题，那么就涉及到对象之间的比较问题。

Java实现对象排序的方式有两种：

  自然排序：java.lang.Comparable
  定制排序：java.util.Comparator

**方式一：自然排序：java.lang.Comparable**

Comparable接口强行对实现它的每个类的对象进行整体排序。这种排序被称为类的自然排序。

实现 Comparable 的类必须实现 compareTo(Object obj) 方法，两个对象即通过 compareTo(Object obj) 方法的返回值来比较大小。

如果当前对象this大于形参对象obj，则返回正整数，

如果当前对象this小于形参对象obj，则返回负整数，

如果当前对象this等于形参对象obj，则返回零。

实现Comparable接口的对象列表（和数组）可以通过 Collections.sort 或Arrays.sort进行自动排序。

实现此接口的对象可以用作有序映射中的键或有序集合中的元素，无需指定比较器。

对于类 C 的每一个 e1 和 e2 来说，当且仅当 e1.compareTo(e2) == 0 与e1.equals(e2) 具有相同的 boolean 值时，类 C 的自然排序才叫做与 equals 一致。建议（虽然不是必需的）最好使自然排序与 equals 一致。

Comparable 的典型实现：(默认都是从小到大排列的)

String：按照字符串中字符的Unicode值进行比较

Character：按照字符的Unicode值来进行比较

数值类型对应的包装类以及BigInteger、BigDecimal：按照它们对应的数值大小进行比较

Boolean：true 对应的包装类实例大于 false 对应的包装类实例

Date、Time等：后面的日期时间比前面的日期时间大


		class Goods implements Comparable {
		    private String name;
		    private double price;
		
		    //按照价格，比较商品的大小
		    @Override
		    public int compareTo(Object o) {
		        if (o instanceof Goods) {
		            Goods other = (Goods) o;
		            if (this.price > other.price) {
		                return 1;
		            } else if (this.price < other.price) {
		                return -1;
		            }
		            return 0;
		        }
		        throw new RuntimeException("输入的数据类型不一致");
		    }
		//构造器、getter、setter、toString()方法略
		}



    public class ComparableTest {
     public static void main(String[] args) {
        Goods[] all = new Goods[4];
        all[0] = new Goods("《红楼梦》", 100);
        all[1] = new Goods("《西游记》", 80);
        all[2] = new Goods("《三国演义》", 140);
        all[3] = new Goods("《水浒传》", 120);
        Arrays.sort(all);
        System.out.println(Arrays.toString(all));
     }
    }

**方式二：定制排序：java.util.Comparator**

-->当元素的类型没有实现java.lang.Comparable接口而又不方便修改代码，或者实现了java.lang.Comparable接口的排序规则不适合当前的操作，那么可以考虑使用 Comparator 的对象来排序，强行对多个对象进行整体排序的比较。

-->重写compare(Object o1,Object o2)方法，比较o1和o2的大小：如果方法返回正整数，则表示o1大于o2；如果返回0，表示相等；返回负整数，表示
o1小于o2。

-->可以将 Comparator 传递给 sort 方法（如 Collections.sort 或 Arrays.sort），从而允许在排序顺序上实现精确控制。

-->还可以使用 Comparator 来控制某些数据结构（如有序 set或有序映射）的顺序，或者为那些没有自然顺序的对象 collection 提供排序。



        Goods[] all = new Goods[4];
        all[0] = new Goods("War and Peace", 100);
        all[1] = new Goods("Childhood", 80);
        all[2] = new Goods("Scarlet and Black", 140);
        all[3] = new Goods("Notre Dame de Paris", 120);
        Arrays.sort(all, new Comparator() {
            @Override
            public int compare(Object o1, Object o2) {
                Goods g1 = (Goods) o1;
                Goods g2 = (Goods) o2;
                return g1.getName().compareTo(g2.getName());
            }
        });
        System.out.println(Arrays.toString(all));

