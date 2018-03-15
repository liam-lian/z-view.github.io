---
layout:     post
title:      python标准模块-itertools
date:  2017-09-19
category:   python标准模块
tags:   [python,python标准模块]
---
itertools 模块提供的迭代器函数有以下几种类型：
- 无限迭代器：生成一个无限序列，比如自然数序列1, 2, 3, 4, ... ；
- 有限迭代器：接收一个或多个序列（sequence）作为参数，进行组合、分组和过滤等；
- 组合生成器：序列的排列、组合，求序列的笛卡儿积等；

### 无限迭代器
####  count(firstval=0, step=1)  
创建一个从firstval (默认值为0) 开始，以step (默认值为1) 为步长的的无限整数迭代器
####  cycle(iterable)  
对iterable 中的元素反复执行循环，返回迭代器  
传入的是一个**iterable的对象**，对这个对象里面的数据进行循环的迭代
####  repeat(object [，times]  
反复生成object，如果给定times，则重复次数为times，否则为无限

### 有限迭代器
####  chain ( iterable 1 ,iterable 2，iterable 3 , . . . )
接收多个可迭代对象作为参数，将它们『连接』起来，作为一个新的迭代器返回  
**chain.from_i terable ( iiterable)**  
接受一个iterable，返回他的迭代器
####  compress ( data , selectors )
对数据进行筛选，当selectors 的某个元素为true 时，则保留data 对应位置的元素，否则去除
####  dropwhile ( predicate , iterable )  
如果predicate(item)为true，则丢弃该元素，否则返回该项及所有后续项
####    takewhile()  
如果predicate(item)为true，则保留该元素，只要predicate(item) 为false，则立即停止迭
####  groupby (iterable [ , keyfunc ] )  
用于对iterable 的连续项进行分组，
如果不指定，则默认对iterable 中的连续相同项进行分组，返回一个(key, sub-iterator) 的迭代
器。
```
data = [ ’ a ’ , ’ bb ’ , ’ cc ’ , ’ ddd ’ , ’ eee ’ , ’ f ’ ]
for key , value_iter in groupby ( data ,len):
输出value_iterator
1:['a']
2:['bb','cc']
3:['ddd','eee']
4:['f']
```



####   ifilter()
####   ifilterfalse()
####   islice()
####   imap()
####  starmap()
####   tee()

####  izip()
####   izip_longest()


