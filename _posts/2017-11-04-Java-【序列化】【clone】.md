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

对象的克隆——clone()方法
---
clone()方法返回的是一个新的引用，使用这个方法相当于在堆上new 出了一个新的实例，新的实例的各个值与原来的实例中的值是一样的。   
clone方法是Object类中的方法，被Protected的修饰。目的是防止被其他的包调用，产生不必要的风险。       
如果你的类允许被别人clone，且需要实现Cloneable接口(是一个标记接口)，如果没有实现这个接口，调用clone()的方法时会抛出异常。   
Object类默认的clone实现是native的，是浅拷贝。对于primitive类型和不可变类型会生成副本。但是对于普通的引用类型，clone之后的对象与原来的实例指向的是同一个位置(引用的值相同)。  
为了实现深拷贝，需要重写clone()方法，例如:
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

对象的序列化
---
序列化是把对象当前的状态存储为字节流，在需要恢复的时候使用读取字节流就能够恢复对象完整的状态。   
序列化的类需要实现Serializable接口(标记接口)。  
序列化需要保存对象的完整状态，因此是一个深拷贝的过程。对于引用类型的变量，会对其继续进行序列化。如果类中的某些变量没有实现Serializable接口，会产生异常。(因此默认是强制深层拷贝的)  
如果某一个对象不希望进行序列化，则可以使用`transient`关键字屏蔽掉。在恢复的时候相应的值赋值为默认值(0,false,null...)
```Java
String savePath="/home/zview/aa.bin";
Person p=new Person(12,"zz",new Address("bj"));
ObjectOutputStream o=new ObjectOutputStream(new FileOutputStream(savePath));
o.writeObject(p);

ObjectInputStream inputStream=new ObjectInputStream(new FileInputStream(savePath));
Person pp= (Person) inputStream.readObject();
```
正式因为对象的序列化是深层拷贝的，因此有些时候可以使用序列化来代替`clone()`实现深层拷贝。
```Java
public static <T> T clone(T obj) throws Exception {
    ByteArrayOutputStream bout = new ByteArrayOutputStream();
    ObjectOutputStream oos = new ObjectOutputStream(bout);
    oos.writeObject(obj);

    ByteArrayInputStream bin = new ByteArrayInputStream(bout.toByteArray());
    ObjectInputStream ois = new ObjectInputStream(bin);
    return (T) ois.readObject();
    // 说明：调用ByteArrayInputStream或ByteArrayOutputStream对象的close方法没有任何意义
    // 这两个基于内存的流只要垃圾回收器清理对象就能够释放资源，这一点不同于对外部资源（如文件流）的释放
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
