---
layout:     post
title:      NIO Buffer
date:  2017-11-20
catagory: NIO
tags:   [NIO]
---
### 基础
一个用来存放特定的基本数据类型的容器 ，在JAVA NIO中缓冲区和通道进行交互  
BUffer常用子类：  
除了boolean之外的7种基本数据类型都有与之相对应的Buffer：  
- ByteBuffer
- CharBuffer
- ShortBuffer
- and so on

#### Buffer中的几个基本属性
- capacity  ==>最大容量
- limit        ==>limit后面的数据不允许操作，position<limit，limit指向的位置不包含数据
- position  ==>当前操作数据的指针的位置
- mark    ==>用来记录一个时刻的position指针的位置    

#### 基本方法
```Java
public static CharBuffer allocate(int capacity) {
        if (capacity < 0)
            throw new IllegalArgumentException();
        return new HeapCharBuffer(capacity, capacity);
}
public final Buffer clear() {
        position = 0;
        limit = capacity;
        mark = -1;
        return this;
    }
public final Buffer flip() {
        limit = position;
        position = 0;
        mark = -1;
        return this;
    }
public final Buffer rewind() {
        position = 0;
        mark = -1;
        return this;
    }
public final Buffer reset() {
        int m = mark;
        if (m < 0)
            throw new InvalidMarkException();
        position = m;
        return this;
}
public final int remaining() {
    return limit - position;
}

```Java
#### 直接缓冲区和非直接缓冲区  
上面的allocate()方法返回的就是非直接缓冲区，缓冲区存在与jvm的用户地址空间中，当需要与本地文件进行IO的时候，数据会在用户地址空间和OS的内核地址空间之间进行靠copy，当需要操作的数据巨大的时候，可以使用直接缓冲区。    
注意：只有ByteBuffer支持直接缓冲区
```
  ByteBuffer bf=ByteBuffer.allocateDirect(100);
  System.out.println(bf.isDirect());  //判断是否是直接缓冲区
```

###高级特性
#### 创建Buffer

```Java
ByteBuffer buffer = ByteBuffer.allocate( 1024 );
byte array[] = new byte[1024];
ByteBuffer buffer = ByteBuffer.wrap( array );
```
#### 缓冲区分片
slice() 方法根据现有的缓冲区创建一种 子缓冲区 。也就是说，它创建一个新的缓冲区，新缓冲区与原来的缓冲区的一部分共享数据。  
缓冲区片对于促进抽象非常有帮助。可以编写自己的函数处理整个缓冲区，而如果想要将这个过程应用于子缓冲区上  
只需取主缓冲区的一个片，并将它传递给您的函数。这比编写自己的函数来取额外的参数以指定要对缓冲区的哪一部分进行操作更容易。  
就是说分片产生一个子缓冲区，他和原来的Buffer公用底层的数组。子缓冲区就是原来的Buffer的position和limit中间的那一段
```Java
    int b[]={1,2,3,4,5,6,7,8};
    IntBuffer intBuffer=IntBuffer.wrap(b);
    intBuffer.position(3);
    intBuffer.limit(5);
    IntBuffer sliceIntBuffer=intBuffer.slice();

    for (int i = 0; i < sliceIntBuffer.limit(); i++) {
        sliceIntBuffer.put(i,sliceIntBuffer.get(i)*10);
    }

    int []res=intBuffer.array();
    for (int i:res)
        System.out.println(i);
```
#####  只读缓冲区
通过调用缓冲区的 asReadOnlyBuffer() 方法，将任何常规缓冲区转换为只读缓冲区，  
这个方法返回一个与原缓冲区完全相同的缓冲区(**并与其共享数据**)，只不过它是只读的。
