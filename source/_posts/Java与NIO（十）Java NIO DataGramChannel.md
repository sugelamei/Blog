---
title: Java与NIO（十）Java NIO DataGramChannel
date: 2019-09-17 21:48:20 
tags: 
    - Java 
    - NIO
---

> 出自：http://ifeve.com/java-nio-all/

Java NIO中的DatagramChannel是一个能收发UDP包的通道。因为UDP是无连接的网络协议，所以不能像其它通道那样读取和写入。它发送和接收的是数据包。


### 打开 DatagramChannel ###

下面是 DatagramChannel 的打开方式：

		//创建DatagramChannel
        DatagramChannel channel = DatagramChannel.open();
        //绑定端口
        channel.bind( new InetSocketAddress(9999));
<!--more-->>>
### 发送数据 ###

通过send()方法从DatagramChannel发送数据，如:

		//创建DatagramChannel
        DatagramChannel channel = DatagramChannel.open();
        //创建缓冲区
        ByteBuffer buffer = ByteBuffer.allocate(128);
        //清除缓冲区
        buffer.clear();
        //将数据放入缓冲区
        buffer.put("发送端发送的数据".getBytes());
        //切换到读模式
        buffer.flip();
        //将buffer中的数据写入channel中发送到指定ip和端口
        channel.send(buffer,new InetSocketAddress("127.0.0.1",9999));

这个例子发送一串字符到”j127.0.0.1”服务器的UDP端口9999。 因为服务端并没有监控这个端口，所以什么也不会发生。也不会通知你发出的数据包是否已收到，因为UDP在数据传送方面没有任何保证。


### 接收数据 ###

通过receive()方法从DatagramChannel接收数据，如：


        //创建DatagramChannel
        DatagramChannel channel = DatagramChannel.open();
        //绑定端口
        channel.bind( new InetSocketAddress(9999));
        //创建缓冲区
        ByteBuffer buffer = ByteBuffer.allocate(128);
        //清除缓冲区
        buffer.clear();
       //从channel接受数据写入buffer
        channel.receive(buffer);
       //打印出接收到的内容
        System.out.println("接收到   ==>" +new String(buffer.array()));

receive()方法会将接收到的数据包内容复制到指定的Buffer. 如果Buffer容不下收到的数据，多出的数据将被丢弃。


### 连接到特定的地址 ###

可以将DatagramChannel“连接”到网络中的特定地址的。由于UDP是无连接的，连接到特定地址并不会像TCP通道那样创建一个真正的连接。而是锁住DatagramChannel ，让其只能从特定地址收发数据。

        //创建DatagramChannel
        DatagramChannel channel = DatagramChannel.open();
        //连接到特定的服务器上
        channel.connect(new InetSocketAddress("127.0.0.1",80));


当连接后，也可以使用read()和write()方法，就像在用传统的通道一样。只是在数据传送方面没有任何保证。这里有几个例子：

            
            channel.read(buf);
            channel.write(but);



