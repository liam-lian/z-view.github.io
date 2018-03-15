---
layout:     post
title:      CharSet与Selector
date:  2017-11-23
catagory:   NIO
tags:   [NIO]
---
### CharSet    
在java.nio.charset包中共提供了Charset、CharsetDecoder、CharsetEncoder、CodeResult、CodingErrorAction五个类，均继承自Object类，其中Charset实现了Comparable接口，其它类均为自身实现。 
  首先应该理解，编码是将String==》bytes，解码是把bytes===》String
```
Charset cs1 = Charset.forName("GBK");
//获取编码器
CharsetEncoder ce = cs1.newEncoder();

//获取解码器
CharsetDecoder cd = cs1.newDecoder();

CharBuffer cBuf = CharBuffer.allocate(1024);
cBuf.put("你好世界！");
cBuf.flip();

//编码
ByteBuffer bBuf = ce.encode(cBuf);

for (int i = 0; i < 12; i++) {
	System.out.println(bBuf.get());
}

//解码
bBuf.flip();
CharBuffer cBuf2 = cd.decode(bBuf);
System.out.println(cBuf2.toString());
```
### Selector
传统阻塞IO如果服务器需要处理多客户端的请求的话，需要采用一下的方式
 ```Java
 @Test
    public void client() throws IOException {
        SocketChannel socketChannel=SocketChannel.open(new InetSocketAddress("127.0.0.1",10001));
        ByteBuffer byteBuffer=ByteBuffer.allocate(1000);
        byteBuffer.put("你好".getBytes());
        byteBuffer.flip();
        socketChannel.write(byteBuffer);
        socketChannel.close();
    }

    @Test
    public void server() throws IOException {
        ServerSocketChannel serverSocketChannel=ServerSocketChannel.open();

        serverSocketChannel.bind(new InetSocketAddress((10001)));

        SocketChannel socketChannel;
        ByteBuffer byteBuffer=ByteBuffer.allocate(1000);
        while((socketChannel=serverSocketChannel.accept())!=null){

            socketChannel.read(byteBuffer);
            byteBuffer.flip();
            System.out.println(new String(byteBuffer.array(),0,byteBuffer.limit()));
            byteBuffer.clear();
            socketChannel.close();
        }
        serverSocketChannel.close();
    }
 ```
 ServerSocket是通过一个while循环来进行监听的，在accept()一个连接成功了之后，假设等待这个连接的数据到来需要较长的时间，那么这个时候服务器端只能等待，即便是有其他的client连接到来，服务器也没有办法进行处理！  
 此处需要引入非阻塞的IO，当需要等待的时候，服务器还可以去监听和处理其他时间，时间被注册到Selector中。  
 例如：  