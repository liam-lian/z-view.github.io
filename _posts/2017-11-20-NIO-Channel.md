---
layout:     post
title:      Channel
date:  2017-11-20
category:   NIO
tags:   [NIO]
---
Channel
----
Channel认为是IO源与IO目标之间的一个连接，但是Channel不能直接操作数据，需要和Buffer进行交互

 #### 主要实现类
 ```
  	java.nio.channels.Channel 接口：
 		|--FileChannel
   	|--SocketChannel
   	|--ServerSocketChannel
  		|--DatagramChannel
 ```
 #### 获取通道的方式
1. Java 针对支持通道的类提供了 getChannel() 方法
     - 本地 IO：
         -  FileInputStream/FileOutputStream
         -  RandomAccessFile
     - 网络IO：
 		- Socket
  		- ServerSocket
  		- DatagramSocket
 2. 在 JDK 1.7 中的 NIO.2 针对各个通道提供了静态方法 open()
 3. 在 JDK 1.7 中的 NIO.2 的 Files 工具类的 newByteChannel()
 #### 通道之间的数据传输
 - transferFrom()
 - transferTo()
#### 内存映射文件
一种利用了虚拟内存的机制。存映射文件允许我们创建和修改那些因为太大而不能放入内存的文件，此时就可以假定整个文件都放在内存中，而且可以完全把它当成非常大的数组来访问。实际上就是把这个文件作为看虚拟内存的一部分。
```Java
 @Test
    public  void test01() throws IOException {

        FileChannel fc1 = FileChannel.open(Paths.get("D:/1.jpg"), StandardOpenOption.READ);
        FileChannel fc2=FileChannel.open(Paths.get("D:/2.jpg"),StandardOpenOption.WRITE,StandardOpenOption.CREATE);
        ByteBuffer bf=ByteBuffer.allocate(100);
        while (fc1.read(bf)!=-1){
            bf.flip();
            fc2.write(bf);
            bf.clear();
        }
        fc1.close();
        fc2.close();
    }

    @Test
    public void test02() throws IOException {

        FileChannel fc1 = FileChannel.open(Paths.get("D:/1.jpg"), StandardOpenOption.READ);
        FileChannel fc2=FileChannel.open(Paths.get("D:/2.jpg"),StandardOpenOption.WRITE,StandardOpenOption.READ,StandardOpenOption.CREATE);

        MappedByteBuffer mbb1=fc1.map(FileChannel.MapMode.READ_ONLY,0,fc1.size());
        MappedByteBuffer mbb2=fc2.map(FileChannel.MapMode.READ_WRITE,0,fc1.size());

        byte c[]=new byte[mbb1.limit()];
        mbb1.get(c);
        mbb2.put(c);
        fc1.close();
        fc2.close();
    }


    @Test
    public void test3() throws IOException{
        FileChannel inChannel = FileChannel.open(Paths.get("d:/1.jpg"), StandardOpenOption.READ);
        FileChannel outChannel = FileChannel.open(Paths.get("d:/2.jpg"), StandardOpenOption.WRITE, StandardOpenOption.READ, StandardOpenOption.CREATE);

		inChannel.transferTo(0, inChannel.size(), outChannel);

        inChannel.close();
        outChannel.close();
    }
```
#### Scatter和Gatter
前者是将Channel中的数据按照顺序写入到多个Buffer中，后者是将多个Buffer中的数据写入到一个Channel中。  
只需要在传入BUffer的位置传入Buffer数组即可，这样数据会依次读入/写入到对应的Buffer中  。

####   SocketChannel & ServerSocketChannel  
```Java
 @Test
    public void client() throws IOException {

        SocketChannel socketChannel=SocketChannel.open(new InetSocketAddress("127.0.0.1",10001));
        ByteBuffer byteBuffer=ByteBuffer.allocate(1000);
        FileChannel fileChannel=FileChannel.open(Paths.get("D:/","a.jpg"), StandardOpenOption.READ);

        while (fileChannel.read(byteBuffer)>0){

            byteBuffer.flip();
            socketChannel.write(byteBuffer);
            byteBuffer.clear();
        }
        socketChannel.close();
        fileChannel.close();
    }

    @Test
    public void server() throws IOException {
        ServerSocketChannel serverSocketChannel=ServerSocketChannel.open();

        serverSocketChannel.bind(new InetSocketAddress((10001)));

        SocketChannel socketChannel=serverSocketChannel.accept();

        FileChannel fileChannel=FileChannel.open(Paths.get("D:/","b.jpg"),StandardOpenOption.WRITE,StandardOpenOption.CREATE);
        ByteBuffer byteBuffer=ByteBuffer.allocate(1000);
        while (socketChannel.read(byteBuffer)>0){
            byteBuffer.flip();
            System.out.println("pap");
            fileChannel.write(byteBuffer);
            byteBuffer.clear();
        }
        serverSocketChannel.close();
        socketChannel.close();
        fileChannel.close();
    }
```
