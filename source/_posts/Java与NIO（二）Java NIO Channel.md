---
title: Java与NIO（二）Java NIO Channel
date: 2019-09-16 23:08:11 
tags: 
    - Java 
    - NIO
---

> 出自：http://ifeve.com/java-nio-all/


Java NIO的通道类似流，但又有些不同：
  
- 既可以从通道中读取数据，又可以写数据到通道。但流的读写通常是单向的。
- 通道可以异步地读写。
- 通道中的数据总是要先读到一个Buffer，或者总是要从一个Buffer中写入。

正如上面所说，从通道读取数据到缓冲区，从缓冲区写入数据到通道。如下图所示：

<!--more-->>>

![](/image/Java与NIO/channel和buffer.png)

### Channel的实现 ###

- FileChannel
- DatagramChannel
- SocketChannel
- ServerSocketChannel


FileChannel 从文件中读写数据。

DatagramChannel 能通过UDP读写网络中的数据。

SocketChannel 能通过TCP读写网络中的数据。

ServerSocketChannel可以监听新进来的TCP连接，像Web服务器那样。对每一个新进来的连接都会创建一个SocketChannel。


### 基本的 Channel 示例 ###

下面是一个使用FileChannel读取数据到Buffer中然后把Buffer中的数据写入FileChannel的示例：


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


        //判断是否读到最后 使用inChannel读取a.txt中的数据写入buffer中
        while ( inChannel.read(buffer)!=-1){
            //将Buffer从写模式切换到读模式（必须调用这个方法）
            buffer.flip();
            //使用outChannel 读取buffer中的数据写入b.txt中
            outChannel.write(buffer);
            //清空buffer
            buffer.clear();

        }
        //使用后记得关闭 不关闭造成资源浪费
        out.close();
        in.close();
        inChannel.close();
        outChannel.close();

    }


注意  buffer.flip() 的调用，首先读取数据到Buffer，然后反转Buffer,接着再从Buffer中读取数据。下一节会深入讲解Buffer的更多细节。




