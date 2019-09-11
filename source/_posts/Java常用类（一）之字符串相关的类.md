---
title: Java 常用类(一)之字符串相关的类
date: 2019-08-10 11:39:36 
tags: Java,常用类
---

### 1.String的特性 ###

--> String类：代表字符串。Java 程序中的所有字符串字面值（如 "abc" ）都作为此类的实例实现。

--> String是一个final类，**代表不可变的字符序列**。

--> 字符串是常量，用双引号引起来表示。**它们的值在创建之后不能更改**。

--> String对象的字符内容是存储在一个字符数组value[]中的。

		public final class String
				implements java.io.Serializable, Comparable<String>, CharSequence {

				/** The value is used for character storage. */
				private final char value[];

				/** Cache the hash code for the string */
				private int hash; // Default to 0

<!--more-->
### 2.String对象的创建 ###

				String str = "hello";

				//本质上this.value = new char[0];
				String s1 = new String(); 

				//this.value = original.value;
				String s2 = new String(String original); 

				//this.value = Arrays.copyOf(value, value.length);
				String s3 = new String(char[] a); 

				String s4 = new String(char[] a,int startIndex,int count);

### 3.字符串的创建过程 ###

   
![](/image/Java常用类（一）之字符串相关的类/String的创建01.png)
![](/image/Java常用类（一）之字符串相关的类/String的创建02.png)

### 4.String str1 = “abc”;与String str2 = new String(“abc”);区别？ ###

 -->**字符串常量存储在字符串常量池，目的是共享**

 -->**字符串非常量对象存储在堆中**


![](/image/Java常用类（一）之字符串相关的类/字符串常量和字符串对象的区别.png)


### 5.字符串对象是如何存储的 ###


		String s1 = "javaEE";

		String s2 = "javaEE";

		String s3 = new String("javaEE");

		String s4 = new String("javaEE");

		System.out.println(s1 == s2);//true

		System.out.println(s1 == s3);//false

		System.out.println(s1 == s4);//false

		System.out.println(s3 == s4);//false

注意：==比较的是地址值

![](/image/Java常用类（一）之字符串相关的类/字符串的存储结构.png)

			Person p1 = new Person();
	        p1.name = "Tom";

	        Person p2 = new Person();
	        p2.name = "Tom";

	        System.out.println(p1.name .equals( p2.name)); //true

	        System.out.println(p1.name == p2.name); //true

	        System.out.println(p1.name == "Tom"); //true

	        String s1 = new String("bcde");
	        String s2 = new String("bcde");

	        System.out.println(s1==s2); //false
注意：equals比较的是地址值

![](/image/Java常用类（一）之字符串相关的类/带String类的对象的存储结构.png)

### 6.字符串的判断 ###

![](/image/Java常用类（一）之字符串相关的类/字符串的判断.png)


**结论：**

  **常量与常量的拼接结果在常量池。且常量池中不会存在相同内容的常量。**

  **只要其中有一个是变量，结果就在堆中**

  **如果拼接的结果调用intern()方法，返回值就在常量池中**


### 7.String使用陷阱 ###

-->String s1 = "a"; 

 **说明：在字符串常量池中创建了一个字面量为"a"的字符串。**

-->s1 = s1 + "b"; 

  **说明：实际上原来的“a”字符串对象已经丢弃了，现在在堆空间中产生了一个字符
串s1+"b"（也就是"ab")。如果多次执行这些改变串内容的操作，会导致大量副本
字符串对象存留在内存中，降低效率。如果这样的操作放到循环中，会极大影响
程序的性能。**

-->String s2 = "ab";

 **说明：直接在字符串常量池中创建一个字面量为"ab"的字符串。**

---> String s3 = "a" + "b";

**说明：s3指向字符串常量池中已经创建的"ab"的字符串。**

-->String s4 = s1.intern();

**说明：堆空间的s1对象在调用intern()之后，会将常量池中已经存在的"ab"字符串赋值给s4**


### 8.使用循环拼接字符串 ###

![](/image/Java常用类（一）之字符串相关的类/使用循环拼接字符串.png)

### 9.以下程序输出什么结果 ###

		public class StringTest {
		
		    String str = new String("good");
		    char[] ch = { 't', 'e', 's', 't' };
		    public void change(String str, char ch[]) {
		        str = "test ok";
		        ch[0] = 'b';
		    }
		    public static void main(String[] args) {
		        StringTest ex = new StringTest();
		        ex.change(ex.str, ex.ch);
		        System.out.print(ex.str + " and ");//good  and
		        System.out.println(ex.ch);//test
		    }
		}

   输出结果为：good and best

   **解释：java值传递中，基本数据类型传的是存储的数据，基本数据类型数组和引用类型传的是地址值；String 不可变性，所以在str传进去的时候，不会改变其值**
        

### 10.String常用方法 ###
         
-->int **length()**：**返回字符串的长度**： return value.length

-->char **charAt(int index)**： **返回某索引处的字符**return value[index]

-->boolean **isEmpty()**：**判断是否是空字符串**：return value.length == 0

-->String **toLowerCase()**：**使用默认语言环境，将 String 中的所有字符转换为小写**

-->String **toUpperCase()**：**使用默认语言环境，将 String 中的所有字符转换为大写**

-->String **trim()**：**返回字符串的副本，忽略前导空白和尾部空白**

-->boolean **equals(Object obj)**：**比较字符串的内容是否相同**

-->boolean **equalsIgnoreCase(String anotherString)**：**与equals方法类似，忽略大小写**

-->String **concat(String str)**：**将指定字符串连接到此字符串的结尾。 等价于用“+”**

-->int **compareTo(String anotherString)**：**比较两个字符串的大小**

-->String **substring(int beginIndex)**：**返回一个新的字符串，它是此字符串的从beginIndex开始截取到最后的一个子字符串**。

-->String **substring(int beginIndex, int endIndex)** ：**返回一个新字符串，它是此字符串从beginIndex开始截取到endIndex(不包含)的一个子字符串。**

-->boolean **endsWith(String suffix)**：**测试此字符串是否以指定的后缀结束**

-->boolean **startsWith(String prefix)**：**测试此字符串是否以指定的前缀开始**

-->boolean **startsWith(String prefix, int toffset)**：**测试此字符串从指定索引开始的子字符串是否以指定前缀开始**

-->boolean **contains(CharSequence s)**：**当且仅当此字符串包含指定的 char 值序列时，返回 true**

-->int **indexOf(String str)**：**返回指定子字符串在此字符串中第一次出现处的索引**

-->int **indexOf(String str, int fromIndex)**：**返回指定子字符串在此字符串中第一次出现处的索引，从指定的索引开始**

-->int **lastIndexOf(String str)**：**返回指定子字符串在此字符串中最右边出现处的索引**

-->int** lastIndexOf(String str, int fromIndex)**：**返回指定子字符串在此字符串中最后一次出现处的索引，从指定的索引开始反向搜索**

**注：indexOf和lastIndexOf方法如果未找到都是返回-1**

-->String **replace(char oldChar, char newChar)**：**返回一个新的字符串，它是通过用 newChar 替换此字符串中出现的所有 oldChar 得到的。**

-->String **replace(CharSequence target, CharSequence replacement)**：**使用指定的字面值替换序列替换此字符串所有匹配字面值目标序列的子字符串**。

-->String **replaceAll(String regex, String replacement)** ： **使用给定的replacement 替换此字符串所有匹配给定的正则表达式的子字符串**。

-->String **replaceFirst(String regex, String replacement)** ： **使用给定的replacement 替换此字符串匹配给定的正则表达式的第一个子字符串**。

-->boolean **matches(String regex)**：**告知此字符串是否匹配给定的正则表达式**。

-->String[] **split(String regex)**：**根据给定正则表达式的匹配拆分此字符串**。

-->String[] **split(String regex, int limit)**：**根据匹配给定的正则表达式来拆分此字符串，最多不超过limit个，如果超过了，剩下的全部都放到最后一个元素中**。



String方法的一些使用：

		 String str = "12hello34world5java7891mysql456";
		//把字符串中的数字替换成,，如果结果中开头和结尾有，的话去掉
		String string = str.replaceAll("\\d+", ",").replaceAll("^,|,$", "");
		System.out.println(string);//hello,world,java,mysql


----------

       	String str = "12345";
		//判断str字符串中是否全部有数字组成，即有1-n个数字组成
		boolean matches = str.matches("\\d+");
		System.out.println(matches);
		String tel = "0571-4534289";
		//判断这是否是一个杭州的固定电话
		boolean result = tel.matches("0571-\\d{7,8}");
		System.out.println(result);
        

----------
		String str = "hello|world|java";
		String[] strs = str.split("\\|");
		for (int i = 0; i < strs.length; i++) {
		System.out.println(strs[i]);
		}
		System.out.println();
		String str2 = "hello.world.java";
		String[] strs2 = str2.split("\\.");
		for (int i = 0; i < strs2.length; i++) {
		System.out.println(strs2[i]);
		}

### 11.String与基本数据类型转换 ###

**字符串-->基本数据类型、包装类**

 -->Integer包装类的public static int parseInt(String s)：可以将由“数字”字符组成的字符串转换为整型。

 -->类似地,使用java.lang包中的Byte、Short、Long、Float、Double类调相应的类方法可以将由“数字”字符组成的字符串，转化为相应的基本数据类型。

**基本数据类型、包装类 -->字符串**

 -->调用String类的public String valueOf(int n)可将int型转换为字符串

 -->相应的valueOf(byte b)、valueOf(long l)、valueOf(float f)、valueOf(double d)、valueOf(boolean b)可由参数的相应类型到字符串的转换


### 12.String与字符数组转换 ###

**字符数组-->字符串**

-->String 类的构造器：String(char[]) 和 String(char[]，int offset，intlength) 分别用字符数组中的全部字符和部分字符创建字符串对象。

**字符串-->字符数组**

-->public char[] toCharArray()：将字符串中的全部字符存放在一个字符数组中的方法。

-->public void getChars(int srcBegin, int srcEnd, char[] dst, int dstBegin)：提供了将指定索引范围内的字符串存放到数组中的方法。

### 13.String与字节数组转换 ###

**字节数组-->字符串**

-->String(byte[])：通过使用平台的默认字符集解码指定的 byte 数组，构造一个新的 String。

-->String(byte[]，int offset，int length) ：用指定的字节数组的一部分，即从数组起始位置offset开始取length个字节构造一个字符串对象。

**字符串-->字节数组**

-->public byte[] getBytes() ：使用平台的默认字符集将此 String 编码为

-->byte 序列，并将结果存储到一个新的 byte 数组中。

-->public byte[] getBytes(String charsetName) ：使用指定的字符集将此 String 编码到 byte 序列，并将结果存储到新的 byte 数组。

  
        String str = "中";

        System.out.println(str.getBytes("ISO8859-1").length);// 1

        System.out.println(str.getBytes("GBK").length);//2

        System.out.println(str.getBytes("UTF-8").length);//3

        System.out.println(new String(str.getBytes("ISO8859-1"),"ISO8859-1"));// 乱码，表示不了中文

        System.out.println(new String(str.getBytes("GBK"), "GBK"));//中

        System.out.println(new String(str.getBytes("UTF-8"), "UTF-8"));//中

### 14.字符串相关的类：StringBuffer ###

-->java.lang.StringBuffer代表可变的字符序列，JDK1.0中声明，可以对字符串内容进行增删，此时不会产生新的对象。
-->很多方法与String相同。
-->作为参数传递时，方法内部可以改变值。

**StringBuffer类不同于String，其对象必须使用构造器生成。有三个构造器：**
-->StringBuffer()：初始容量为16的字符串缓冲区
-->StringBuffer(int size)：构造指定容量的字符串缓冲区
-->StringBuffer(String str)：将内容初始化为指定字符串内容


**StringBuffer类的常用方法**

  -->StringBuffer **append**(xxx)：提供了很多的append()方法，用于进行字符串拼接

  -->StringBuffer **delete**(int start,int end)：删除指定位置的内容

  -->StringBuffer **replace**(int start, int end, String str)：把[start,end)位置替换为str

  -->StringBuffer **insert**(int offset, xxx)：在指定位置插入xxx

  -->StringBuffer **reverse**() ：把当前字符序列逆转

**注意：**

  当append和insert时，如果原来value数组长度不够，可扩容。

  如上这些方法支持方法链操作。

 

此外，还定义了如下的方法： 


-->public int **indexOf**(String str)

-->public String **substring**(int start,int end)

-->public int **length**()

-->public char **charAt**(int n )

-->public void **setCharAt**(int n ,char ch)


### 15.字符串相关的类：StringBuilder ###

StringBuilder 和 StringBuffer 非常类似，均代表可变的字符序列，而且提供相关功能的方法也一样


**对比String、StringBuffer、StringBuilder**

-->String(JDK1.0)：不可变字符序列,final char[]储存

-->StringBuffer(JDK1.0)：可变字符序列、效率低、线程安全， char[]储存

-->StringBuilder(JDK 5.0)：可变字符序列、效率高、线程不安全， char[]储存

注意：作为参数传递的话，方法内部String不会改变其值，StringBuffer和StringBuilder会改变其值。

性能对比：

		long startTime = 0L;
		long endTime = 0L;
		String text = "";
		StringBuffer buffer = new StringBuffer("");
		StringBuilder builder = new StringBuilder("");
		//开始对比
		startTime = System.currentTimeMillis();
		for (int i = 0; i < 20000; i++) {
		buffer.append(String.valueOf(i));
		}
		endTime = System.currentTimeMillis();
		System.out.println("StringBuffer的执行时间：" + (endTime - startTime));
		startTime = System.currentTimeMillis();
		for (int i = 0; i < 20000; i++) {
		builder.append(String.valueOf(i));
		}
		endTime = System.currentTimeMillis();
		System.out.println("StringBuilder的执行时间：" + (endTime - startTime));
		startTime = System.currentTimeMillis();
		for (int i = 0; i < 20000; i++) {
		text = text + i;
		}
		endTime = System.currentTimeMillis();
		System.out.println("String的执行时间：" + (endTime - startTime));
		      
   输出结果：

   StringBuffer的执行时间：7

   StringBuilder的执行时间：2

   String的执行时间：1417

----------


		String str = null;

        StringBuffer sb = new StringBuffer();

        sb.append(str);

        System.out.println(sb.length());//4

        System.out.println(sb);//null

        StringBuffer sb1 = new StringBuffer(str);

        System.out.println(sb1);//java.lang.NullPointerException



