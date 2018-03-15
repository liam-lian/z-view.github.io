---
layout:     post
title:      Optional
date:  2017-12-05
category:   Java8新特性
tags:   [Java8新特性]
---
  OPtional是Java 8中的新特性，在java.util包中。javadoc描述为
 ```
 这是一个可以为null的容器对象。如果值存在则isPresent()方法会返回true，调用get()方法会返回该对象。
 ```
#### 构造方法  
```
Optional.of(obj)  //obj为空时抛出异常
Optional.ofNullable(obj) //obj为空调用Optional.empty() ，否则调用Optional.of(obj)
Optional.empty()  //创建一个空的Optional(就是里面存的值是null)
```
#### 其他方法
```
- isPresent()        如果值存在返回true，否则返回false
- get()              如果Optional有值则将其返回，否则抛出NoSuchElementException
- ifPresent(Consumer<? super T> consumer)如果Optional实例有值则为其调用consumer，否则不做处理
- orElse(T other)     如果有值则将其返回，否则返回指定的其它值。
- orElseGet(Supplier<? extends T> other)   接受Supplier接口的实现用来生成默认值
- orElseThrow(Supplier<? extends X> exceptionSupplier)  传入一个lambda表达式或方法，如果值不存在来抛出异常
- map(Function<? super T, ? extends U> mapper)如果有值，则对其执行调用mapping函数得到返回值。
                    如果返回值不为null，则创建包含mapping返回值的Optional作为map方法返回值，否则返回空Optional。
- flatMap(Function<? super T, Optional<U>> mapper) 如果有值，为其执行mapping函数返回Optional类型返回值，否则返回空Optional。
flatMap与map（Funtion）方法类似，区别在于flatMap中的mapper返回值必须是Optional。调用结束时，flatMap不会对结果用Optional封装
- filter(Predicate<? super T> predicate)  如果有值并且满足断言条件返回包含该值的Optional，否则返回空Optional。
```
#### 总结
使用 Optional 时尽量不直接调用 Optional.get() 方法, Optional.isPresent() 更应该被视为一个私有方法,   应依赖于其他像 Optional.orElse() , Optional.orElseGet() , Optional.map() 等这样的方法  
几个应用： 

1. 存在返回，否则填充默认值
```
return user.orElse(user::new); 
```
2. 存在进行处理转换，否则返回空OPtional
```
return user.map(u -> u.getUsername())
           .map(name -> name.toUpperCase())
           .orElse(null);
```
3. 满足条件返回，不满足返回空Optional  =>filter
4. 非空处理，否则忽视  ifPresent
```
Optional o=Optional.ofNullable("abc").map(a->a.toUpperCase()).map(a->a.charAt(0)).filter(x->x=='B');
        o.ifPresent(System.out::println);
```

