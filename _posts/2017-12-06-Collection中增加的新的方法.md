---
layout:     post
title:      Java8新特性-Collection中增加的新的方法
date:  2017-12-06
keywords:   Java8新特性
category:   Java8新特性
tags:   [Java,Java8新特性]
---
#### Collection中的新方法  
1. forEach()  
```
list.forEach( str -> {
        if(str.length()>3)
            System.out.println(str);
    });
```
2. removeIf()
```
list.removeIf(str -> str.length()>3); // 删除长度大于3的元素
```
3. replaceAll()
```
list.replaceAll(str -> {
    if(str.length()>3)
        return str.toUpperCase();
    return str;
});
```
4. sort()  
```
list.sort((str1, str2) -> str1.length()-str2.length());
```
5. stream()和parallelStream()
#### Map的新方法
1. forEach()  
```
map.forEach((k, v) -> System.out.println(k + "=" + v));
```
2. getOrDefault()  
该方法跟Lambda表达式没关系，但是很有用。方法签名为V getOrDefault(Object key, V defaultValue)，作用是按照给定的key查询Map中对应的value，如果没有找到则返回defaultValue。  
```
System.out.println(map.getOrDefault(4, "NoValue"));
```
3. putIfAbsent()  
V putIfAbsent(K key, V value)，作用是只有在不存在key值的映射或映射值为null时，才将value指定的值放入到Map中，否则不对Map做更改．该方法将条件判断和赋值合二为一，使用起来更加方便．  
4. remove()  
remove(Object key, Object value)方法，只有在当前Map中**key正好映射到value时**才删除该映射，否则什么也不做．  
5. replace()  
- replace(K key, V value)，只有在当前Map中**key的映射存在时**才用value去替换原来的值，否则什么也不做．
- replace(K key, V oldValue, V newValue)，只有在当前Map中**key的映射存在且等于oldValue时**才用newValue去替换原来的值，否则什么也不做．
6. replaceAll()  
```
map.replaceAll((k, v) -> v.toUpperCase());
```
7.


