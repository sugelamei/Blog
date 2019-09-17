---
title: Java与NIO（五）Java NIO 通道之间的数据传输
date: 2019-09-17 14:32:55 
tags: 
    - Java 
    - NIO
---

> 出自：http://ifeve.com/java-nio-all/

在Java NIO中，如果两个通道中有一个是FileChannel，那你可以直接将数据从一个channel（译者注：channel中文常译作通道）传输到另外一个channel。

#### transferFrom() ####


FileChannel的transferFrom()方法可以将数据从源通道传输到FileChannel中（译者注：这个方法在JDK文档中的解释为将字节从给定的可读取字节通道传输到此通道的文件中）。下面是一个简单的例子：

       //创建文件输入流
        FileInputStream in = new FileInputStream(new File("a.txt"));
        //创建文件输出流
        FileOutputStream out   = new FileOutputStream(new File("b.txt"));
        //创建缓冲区
        ByteBuffer buffer  = ByteBuffer.allocate(24);
        //获得输入channel
        FileChannel inChannel  = in.getChannel();
        //获取输出channel
        FileChannel outChannel = out.getChannel();

        //从inChannel中将数据写入到outChannel中
        outChannel.transferFrom(inChannel,0,inChannel.size());

        //使用后记得关闭 不关闭造成资源浪费
        out.close();
        in.close();
        inChannel.close();
        outChannel.close();
<!--more-->>>

方法的输入参数position表示从position处开始向目标文件写入数据，count表示最多传输的字节数。如果源通道的剩余空间小于 count 个字节，则所传输的字节数要小于请求的字节数。

transferTo()方法将数据从FileChannel传输到其他的channel中。下面是一个简单的例子：

        //创建文件输入流
        FileInputStream in = new FileInputStream(new File("a.txt"));
        //创建文件输出流
        FileOutputStream out   = new FileOutputStream(new File("b.txt"));
        //创建缓冲区
        ByteBuffer buffer  = ByteBuffer.allocate(24);
        //获得输入channel
        FileChannel inChannel  = in.getChannel();
        //获取输出channel
        FileChannel outChannel = out.getChannel();

        //将inChannel中数据写入到outChannel
        inChannel.transferTo(0,outChannel.size(),outChannel);

        //使用后记得关闭 不关闭造成资源浪费
        out.close();
        in.close();
        inChannel.close();
        outChannel.close();


是不是发现这个例子和前面那个例子特别相似？除了调用方法的FileChannel对象不一样外，其他的都一样。
上面所说的关于SocketChannel的问题在transferTo()方法中同样存在。SocketChannel会一直传输数据直到目标buffer被填满。