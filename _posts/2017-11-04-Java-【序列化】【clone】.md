---
layout:     post
title:      【序列化】【clone】
date:  2017-11-04
category:   Java
tags:   [Java]
---

深拷贝和浅拷贝
---
首先对于基本数据类型不再考虑的范围之内，对于基本数据类型是直接赋值拷贝的，主从对象中完全独立
对于引用类型:
- 浅拷贝指的是拷贝出来的对象和原来的对象中的引用类型指向的仍然是堆上同一个位置，二者是互相联动的
- 深拷贝指的是拷贝出来的对象和原来的对象中的引用类型独立，不仅仅拷贝了基本数据类型还执行了其中的引用对象的clone()方法，二者完全独立。

clone()方法
---
clone方法是Object类中的方法，被Protected的修饰。目的是防止被其他的包调用，产生不必要的风险。       
如果你的类允许被别人clone，应该重写clone()方法，将权限扩展为public，并且需要实现Cloneable接口，
这个接口是一个空的标记接口，如果没有实现这个接口，调用clone()的方法时会抛出异常。
Object类默认的clone实现是native的，是浅拷贝，需要我们重写方法，调用每一个引用类型的clone以实现深拷贝。   
例如:
```Java
Class Book implememt Cloneable{
  int num;
  Author author;
  public Object clone(){
    Object clone=super.clone();
    //这里实现了author的clone，至于author是不是深拷贝的,
    //需要看author中的clone()方法
    clone.author=author.clone();
    return clone;
  }
}
```

serialVersionUID[序列化--与clone无关]
---
serialVersionUID的作用主要来自于安全性检查！  
Java的序列化机制是通过在运行时判断类的serialVersionUID来验证版本一致性的。在进行反序列化时，JVM会把传来的字节流中的serialVersionUID与本地相应实体（类）的serialVersionUID进行比较，如果相同就认为是一致的，可以进行反序列化，否则就会出现序列化版本不一致的异常。  

serialVersionUID如果没有自己设置，会由编译的时候根据当前的类名等信息生成一个（摘要算法），这样一旦这个类发生了改变（即便只是增加了一个空格），就不能反序列化了，导致兼容性的问题。  
比如如果没有serialVersionUID，java v1.5中有一个类，在v1.6的时候增加了一个方法。如果在1.5时序列化的对象，在1.6的时候是无法序列化回来的，  
serialVersionUID有两种显示的生成方式：         
一个是默认的1L，比如：  
`private static final long serialVersionUID = 1L;`         
一个是根据类名、接口名、成员方法及属性等来生成一个64位的哈希字段，比如   
`private static final   long     serialVersionUID = xxxxL`

注意点
---
1. 只有String，或数组，或Enum，或Serializable可以序列化，因此被序列化的对象的所有属性都需要是以上的四种之一
2. transient关键字修饰的字段，在序列化的时候会被忽略
