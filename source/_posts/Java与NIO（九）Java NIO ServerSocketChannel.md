---
title: Java与NIO（九）Java NIO ServerSocketChannel
date: 2019-09-17 20:48:48  
tags: 
    - Java 
    - NIO
---

> 出自：http://ifeve.com/java-nio-all/

Java NIO中的 ServerSocketChannel 是一个可以监听新进来的TCP连接的通道, 就像标准IO中的ServerSocket一样。ServerSocketChannel类在 java.nio.channels包中。

### 打开 ServerSocketChannel ###

通过调用 ServerSocketChannel.open() 方法来打开ServerSocketChannel.如：

        //创建一个ServerSocketChannel
        ServerSocketChannel channel =  ServerSocketChannel.open();
        //和端口绑定 客户端可以通过ip和端口进行通信
        channel.socket().bind(new InetSocketAddress(8080));

### 关闭 ServerSocketChannel ###

        //关闭 ServerSocketChannel
        channel.close();
<!--more-->>>
### 监听新进来的连接 ###

通过 ServerSocketChannel.accept() 方法监听新进来的连接。当 accept()方法返回的时候,它返回一个包含新进来的连接的 SocketChannel。因此, accept()方法会一直阻塞到有新连接到达。

通常不会仅仅只监听一个连接,在while循环中调用 accept()方法. 如下面的例子：

			while(true){
			    SocketChannel socketChannel =
			            serverSocketChannel.accept();
			
			    //do something with socketChannel...
			}

当然,也可以在while循环中使用除了true以外的其它退出准则。


### 非阻塞模式 ###

ServerSocketChannel可以设置成非阻塞模式。在非阻塞模式下，accept() 方法会立刻返回，如果还没有新进来的连接,返回的将是null。 因此，需要检查返回的SocketChannel是否是null.如：

			ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
			
			serverSocketChannel.socket().bind(new InetSocketAddress(9999));
			serverSocketChannel.configureBlocking(false);
			
			while(true){
			    SocketChannel socketChannel =
			            serverSocketChannel.accept();
			
			    if(socketChannel != null){
			        //do something with socketChannel...
			    }
			}


### 总结 ###


#### 服务端代码 ####
注意：启动先启动服务端，然后启动客户端

       //创建一个ServerSocketChannel
        ServerSocketChannel channel = ServerSocketChannel.open();
        //和端口绑定 客户端可以通过ip和端口进行通信
        channel.socket().bind(new InetSocketAddress(8080));
        //获取SocketChannel
        SocketChannel accept = channel.accept();
        //创建缓冲区
        ByteBuffer buffer = ByteBuffer.allocate(128);
        //将channel读取然后写入buffer中
        accept.read(buffer);
        //打印出接收到信息：
        System.out.println("服务器接收到信息："+new String(buffer.array()));

        //关闭 ServerSocketChannel
        channel.close();


#### 客户端代码 ####


        //获取SocketChannel
        SocketChannel channel = SocketChannel.open();
        //设置channel为非阻塞模式
        channel.configureBlocking(false);
        //连接到指定ip
        channel.connect(new InetSocketAddress("127.0.0.1", 8080));

        //判断连接是否完成
        if (channel.finishConnect()) {
            //创建缓冲区
            ByteBuffer buffer = ByteBuffer.allocate(128);

            //清除buffer中的数据
             buffer.clear();

             buffer.put("测试客户端".getBytes());

            //将buffer从写模式切换到读模式
            buffer.flip();
            //读取buffer中的数据写入channel中
            channel.write(buffer);

            //关闭SocketChannel
            channel.close();
        }

