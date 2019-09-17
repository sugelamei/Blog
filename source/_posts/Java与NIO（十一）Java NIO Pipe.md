---
title: Java与NIO（十一）Java NIO Pipe
date: 2019-09-17 20:48:48  
tags: 
    - Java 
    - NIO
---

> 出自：http://ifeve.com/java-nio-all/

Java NIO 管道是2个线程之间的单向数据连接。Pipe有一个source通道和一个sink通道。数据会被写到sink通道，从source通道读取。

这里是Pipe原理的图示：

![](/image/Java与NIO/pipe.png)


### 创建管道 ###

通过Pipe.open()方法打开管道。例如：


		//创建pipe
        Pipe pipe =  Pipe.open();
<!--more-->>>
### 向管道写数据 ###

要向管道写数据，需要访问sink通道。像这样：

       //获取sink通道（写数据）
        Pipe.SinkChannel sink = pipe.sink();

通过调用SinkChannel的write()方法，将数据写入sink,像这样：


		//获取sink通道（写数据）
        Pipe.SinkChannel sink = pipe.sink();
        buffer.put("sugelamei".getBytes());
        buffer.flip();
        sink.write(buffer);

### 从管道读取数据 ###

从读取管道的数据，需要访问source通道，像这样：

		Pipe.SourceChannel source = pipe.source();

调用source通道的read()方法来读取数据，像这样：

        //获取source通道（读数据）
        Pipe.SourceChannel source = pipe.source();
        source.read(buffer1);
        System.out.println(new String(buffer1.array()));

read()方法返回的int值会告诉我们多少字节被读进了缓冲区。