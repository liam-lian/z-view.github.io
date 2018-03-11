---
layout:     post
title:      NIO Buffer
date:  2017-11-20
catagory: NIO
tags:   [NIO]
---
Buffer
---
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
```
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

```
#### 直接缓冲区和非直接缓冲区  
上面的allocate()方法返回的就是非直接缓冲区，缓冲区存在与jvm的用户地址空间中，当需要与本地文件进行IO的时候，数据会在用户地址空间和OS的内核地址空间之间进行靠copy，当需要操作的数据巨大的时候，可以使用直接缓冲区。    
注意：只有ByteBuffer支持直接缓冲区
```
  ByteBuffer bf=ByteBuffer.allocateDirect(100);
  System.out.println(bf.isDirect());  //判断是否是直接缓冲区
```




















