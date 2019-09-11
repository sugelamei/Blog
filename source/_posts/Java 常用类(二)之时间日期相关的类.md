---
title: Java常用类(二)之时间日期相关的类
date: 2019-08-11 14:33:17  
tags: 
    - Java 
    - 常用类
---

### JDK8之前日期时间API ###

时间相关的API总览：

![](/image/Java常用类(二)之时间日期相关的类/时间日期类总览.png)

**1. java.lang.System类**

   System类提供的public static long currentTimeMillis()用来返回当前时间与1970年1月1日0时0分0秒之间以毫秒为单位的时间差。

   此方法适于计算时间差。

计算世界时间的主要标准有：
   
   UTC(Coordinated Universal Time)
   GMT(Greenwich Mean Time)
   CST(Central Standard Time)


**2. java.util.Date类**

  表示特定的瞬间，精确到毫秒

构造器：
   Date()：使用无参构造器创建的对象可以获取本地当前时间。
   Date(long date)
   常用方法
   getTime():返回自 1970 年 1 月 1 日 00:00:00 GMT 以来此 Date 对象表示的毫秒数。
   toString():把此 Date 对象转换为以下形式的 String： dow mon ddhh:mm:ss zzz yyyy 其中： dow 是一周中的某一天 (Sun, Mon, Tue, 
Wed, Thu, Fri, Sat)，zzz是时间标准。其它很多方法都过时了

		import java.util.Date;

		Date date = new Date();
		System.out.println(date);

		System.out.println(System.currentTimeMillis());
		System.out.println(date.getTime());

		Date date1 = new Date(date.getTime());

		System.out.println(date1.getTime());
		System.out.println(date1.toString());


**3. java.text.SimpleDateFormat类**

Date类的API不易于国际化，大部分被废弃了，java.text.SimpleDateFormat
类是一个不与语言环境有关的方式来格式化和解析日期的具体类。

格式化：**日期-->文本**

解析：**文本-->日期**

格式化：
SimpleDateFormat() ：默认的模式和语言环境创建对象
public SimpleDateFormat(String pattern)：该构造方法可以用参数pattern指定的格式创建一个对象，该对象调用：
public String format(Date date)：方法格式化时间对象date

解析：
public Date parse(String source)：从给定字符串的开始解析文本，以生成一个日期。


pattern中各个字母代表的含义如下图：

比如：yyyyMMdd 代表20190811(2019年8月11日)

![](/image/Java常用类(二)之时间日期相关的类/格式中字母代表的含义.png)


		Date date = new Date(); // 产生一个Date实例

		// 产生一个formater格式化的实例
		SimpleDateFormat formater = new SimpleDateFormat();

		System.out.println(formater.format(date));// 打印输出默认的格式

		SimpleDateFormat formater2 = new SimpleDateFormat("yyyy年MM月dd日 EEE 
		HH:mm:ss");

		System.out.println(formater2.format(date));

		try {
		// 实例化一个指定的格式对象
		Date date2 = formater2.parse("2008年08月08日 星期一 08:08:08");

		// 将指定的日期解析后格式化按指定的格式输出
		System.out.println(date2.toString());
		} catch (ParseException e) {
		e.printStackTrace();
		}


**4. java.util.Calendar(日历)类**

Calendar是一个抽象基类，主用用于完成日期字段之间相互操作的功能。

获取Calendar实例的方法

-->使用Calendar.getInstance()方法
-->调用它的子类GregorianCalendar的构造器。

一个Calendar的实例是系统时间的抽象表示，通过get(int field)方法来取得想要的时间信息。
比如YEAR、MONTH、DAY_OF_WEEK、HOUR_OF_DAY 、MINUTE、SECOND

-->public void set(int field,int value)
-->public void add(int field,int amount)
-->public final Date getTime()
-->public final void setTime(Date date)

注意:

-->获取月份时：一月是0，二月是1，以此类推，12月是11
-->获取星期时：周日是1，周二是2 ， 。。。。周六是7

	Calendar calendar = Calendar.getInstance();

	// 从一个 Calendar 对象中获取 Date 对象
	Date date = calendar.getTime();

	// 使用给定的 Date 设置此 Calendar 的时间
	date = new Date(234234235235L);
	calendar.setTime(date);

	calendar.set(Calendar.DAY_OF_MONTH, 8);
	System.out.println("当前时间日设置为8后,时间是:" + calendar.getTime());

	calendar.add(Calendar.HOUR, 2);
	System.out.println("当前时间加2小时后,时间是:" + calendar.getTime());

	calendar.add(Calendar.MONTH, -2);
	System.out.println("当前日期减2个月后,时间是:" + calendar.getTime());


### JDK8中新日期时间API ###

**新日期时间API出现的背景**

如果我们可以跟别人说：“我们在1502643933071见面，别晚了！”那么就再简单不过了。但是我们希望时间与昼夜和四季有关，于是事情就变复杂了。JDK 1.0中包含了一个java.util.Date类，但是它的大多数方法已经在JDK 1.1引入Calendar类之后被弃用了。而Calendar并不比Date好多少。它们面临的问题是：

**可变性：像日期和时间这样的类应该是不可变的。**

**偏移性：Date中的年份是从1900开始的，而月份都从0开始。**

**格式化：格式化只对Date有用，Calendar则不行。**

**此外，它们也不是线程安全的；不能处理闰秒等。**

总结：**对日期和时间的操作一直是Java程序员最痛苦的地方之一。**



**新时间日期API**

第三次引入的API是成功的，并且Java 8中引入的java.time API 已经纠正了过去的缺陷，将来很长一段时间内它都会为我们服务。Java 8 吸收了 Joda-Time 的精华，以一个新的开始为 Java 创建优秀的 API。新的 java.time 中包含了所有关于**本地日期（LocalDate）、本地时间（LocalTime）、本地日期时间（LocalDateTime）、时区（ZonedDateTime）和持续时间（Duration）**的类。历史悠久的 Date 类新增了**toInstant()** 方法，用于把 Date 转换成新的表示形式。这些新增的本地化时间日期 API 大大简化了日期时间和本地化的管理。

**java.time **– 包含值对象的基础包
**java.time.chrono** – 提供对不同的日历系统的访问
**java.time.format** – 格式化和解析时间和日期
**java.time.temporal **– 包括底层框架和扩展特性
**java.time.zone** – 包含时区支持的类

说明：大多数开发者只会用到基础包和format包，也可能会用到temporal包。因此，尽管有68个新的公开类型，大多数开发者，大概将只会用到其中的三分之一。



**LocalDate、LocalTime、LocalDateTime**

LocalDate、LocalTime、LocalDateTime 类是其中较重要的几个类，它们的实例是不可变的对象，分别表示使用 ISO-8601日历系统的日期、时间、日期和时间。
它们提供了简单的本地日期或时间，并不包含当前的时间信息，也不包含与时区相关的信息。

-->LocalDate代表IOS格式（yyyy-MM-dd）的日期,可以存储 生日、纪念日等日期。

-->LocalTime表示一个时间，而不是日期。

-->LocalDateTime是用来表示日期和时间的，这是一个最常用的类之一。

注：ISO-8601日历系统是国际标准化组织制定的现代公民的日期和时间的表示
法，也就是公历。

**now() / * now(ZoneId zone)** 静态方法，根据当前时间创建对象/指定时区的对象

**of()** 静态方法，根据指定日期/时间创建对象

**getDayOfMonth()/getDayOfYear()** 获得月份天数(1-31) /获得年份天数(1-366)

**getDayOfWeek() **获得星期几(返回一个 DayOfWeek 枚举值)

**getMonth()** 获得月份, 返回一个 Month 枚举值

**getMonthValue() / getYear()** 获得月份(1-12) /获得年份

**getHour()/getMinute()/getSecond()** 获得当前对象对应的小时、分钟、秒

**withDayOfMonth()/withDayOfYear()/withMonth()/withYear()** 将月份天数、年份天数、月份、年份修改为指定的值并返回新的对象

**plusDays(), plusWeeks(), plusMonths(), plusYears(),plusHours()** 向当前对象添加几天、几周、几个月、几年、几小时

**minusMonths() / minusWeeks()/inusDays()/miYears()/minusHours()** 从当前对象减去几月、几周、几天、几年、几小时



** 瞬时：Instant**

-->Instant：时间线上的一个瞬时点。 这可能被用来记录应用程序中的事件时间戳。

-->在处理时间和日期的时候，我们通常会想到年,月,日,时,分,秒。然而，这只是时间的一个模型，是面向人类的。第二种通用模型是面向机器的，或者说是连续的。在此模型中，时间线中的一个点表示为一个很大的数，这有利于计算机处理。在UNIX中，这个数从1970年开始，以秒为的单位；同样的，在Java中，也是从1970年开始，但以毫秒为单位。

-->java.time包通过值类型Instant提供机器视图，不提供处理人类意义上的时间单位。Instant表示时间线上的一点，而不需要任何上下文信息，例如，时区。概念上讲，它只是简单的表示自1970年1月1日0时0分0秒（UTC）开始的秒数。因为java.time包是基于纳秒计算的，所以Instant的精度可以达到纳秒级。(1 ns = 10-9 s) 1秒 = 1000毫秒 =10^6微秒=10^9纳秒

**now()** 静态方法，返回默认UTC时区的Instant类的对象

**ofEpochMilli(long epochMilli)** 静态方法，返回在1970-01-01 00:00:00基础上加上指定毫秒数之后的Instant类的对象

**atOffset(ZoneOffset offset)** 结合即时的偏移来创建一个OffsetDateTime

** OffsetDateTimetoEpochMilli()** 返回1970-01-01 00:00:00到当前时间的毫秒数，即为时间戳

**特别注意**：**时间戳是指格林威治时间1970年01月01日00时00分00秒(北京时间1970年01月01日08时00分00秒)起至现在的总秒数。**


**格式化与解析日期或时间**

java.time.format.DateTimeFormatter 类：该类提供了三种格式化方法：

预定义的标准格式。

如：ISO_LOCAL_DATE_TIME;ISO_LOCAL_DATE;ISO_LOCAL_TIME

本地化相关的格式。

如：ofLocalizedDateTime(FormatStyle.LONG)

自定义的格式。

如：ofPattern(“yyyy-MM-dd hh:mm:ss”)


**ofPattern(String pattern)** 静态方法 ， 返 回一个指定字符串格式的DateTimeFormatter

**format(TemporalAccessor t)** 格式化一个日期、时间，返回字符串

**parse(CharSequence text)** 将指定格式的字符序列解析为一个日期、时间


**其它API**

-->ZoneId：该类中包含了所有的时区信息，一个时区的ID，如 Europe/Paris

-->ZonedDateTime：一个在ISO-8601日历系统时区的日期时间，如 2007-12-03T10:15:30+01:00 Europe/Paris。

-->其中每个时区都对应着ID，地区ID都为“{区域}/{城市}”的格式，例如：Asia/Shanghai等

-->Clock：使用时区提供对当前即时、日期和时间的访问的时钟。

-->持续时间：Duration，用于计算两个“时间”间隔

-->日期间隔：Period，用于计算两个“日期”间隔

-->TemporalAdjuster : 时间校正器。有时我们可能需要获取例如：将日期调整到“下一个工作日”等操作。

-->TemporalAdjusters : 该类通过静态方法(firstDayOfXxx()/lastDayOfXxx()/nextXxx())提供了大量的常用TemporalAdjuster 的实现。



			//ZoneId:类中包含了所有的时区信息
			// ZoneId的getAvailableZoneIds():获取所有的ZoneId
			Set<String> zoneIds = ZoneId.getAvailableZoneIds();
			for (String s : zoneIds) {
			System.out.println(s);
			}

			// ZoneId的of():获取指定时区的时间
			LocalDateTime localDateTime = LocalDateTime.now(ZoneId.of("Asia/Tokyo"));
			System.out.println(localDateTime);

			//ZonedDateTime:带时区的日期时间
			// ZonedDateTime的now():获取本时区的ZonedDateTime对象
			ZonedDateTime zonedDateTime = ZonedDateTime.now();
			System.out.println(zonedDateTime);

			// ZonedDateTime的now(ZoneId id):获取指定时区的ZonedDateTime对象
			ZonedDateTime zonedDateTime1 = ZonedDateTime.now(ZoneId.of("Asia/Tokyo"));
			System.out.println(zonedDateTime1);

			//Duration:用于计算两个“时间”间隔，以秒和纳秒为基准
			LocalTime localTime = LocalTime.now();
			LocalTime localTime1 = LocalTime.of(15, 23, 32);

			//between():静态方法，返回Duration对象，表示两个时间的间隔
			Duration duration = Duration.between(localTime1, localTime);
			System.out.println(duration);
			System.out.println(duration.getSeconds());
			System.out.println(duration.getNano());
			LocalDateTime localDateTime = LocalDateTime.of(2016, 6, 12, 15, 23, 32);
			LocalDateTime localDateTime1 = LocalDateTime.of(2017, 6, 12, 15, 23, 32);
			Duration duration1 = Duration.between(localDateTime1, localDateTime);
			System.out.println(duration1.toDays());

			//Period:用于计算两个“日期”间隔，以年、月、日衡量
			LocalDate localDate = LocalDate.now();
			LocalDate localDate1 = LocalDate.of(2028, 3, 18);
			Period period = Period.between(localDate, localDate1);
			System.out.println(period);
			System.out.println(period.getYears());
			System.out.println(period.getMonths());
			System.out.println(period.getDays());
			Period period1 = period.withYears(2);
			System.out.println(period1);

			// TemporalAdjuster:时间校正器
			// 获取当前日期的下一个周日是哪天？
			TemporalAdjuster temporalAdjuster = TemporalAdjusters.next(DayOfWeek.SUNDAY);
			LocalDateTime localDateTime = LocalDateTime.now().with(temporalAdjuster);
			System.out.println(localDateTime);

			// 获取下一个工作日是哪天？
			LocalDate localDate = LocalDate.now().with(new TemporalAdjuster() {
			@Override
			public Temporal adjustInto(Temporal temporal) {
			LocalDate date = (LocalDate) temporal;
			if (date.getDayOfWeek().equals(DayOfWeek.FRIDAY)) {

			return date.plusDays(3);

			} else if (date.getDayOfWeek().equals(DayOfWeek.SATURDAY)) {

			return date.plusDays(2);

			} else {

			return date.plusDays(1);

			     }
			   }
			});
			System.out.println("下一个工作日是：" + localDate);


**与传统日期处理的转换**

![](/image/Java常用类(二)之时间日期相关的类/新老日期处理转化.png)








