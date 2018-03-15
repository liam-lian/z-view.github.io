---
layout:     post
title:      python标准模块-collections
date:  2017-09-19
category:   python标准模块
tags:   [python,python标准模块]
---
# collections

collections提供了5个高性能的数据模型
## Counter
Counter用来统计字符串、列表等元素的个数
```python
from collections import Counter
str='zzzsadaasaskldas'
c=Counter(str)
c.most_common(2) #获得列表中的出现次数最多的两个字符
输出[('a', 5), ('s', 4)]
```
Counter实际上是dict的子类
同时两个Counter之间还可以进行运算
- c1 + c2
- c1 - c2
- c1 & c2            
c1、c2中都有并且取较小值
- c1 | c2  
c1、c2的元素的并集，如果都有，选择数值较大的

## OrderedDict
python中的 Dict是无序的（Hash散列的），这个数据模型将字典按照插入的顺序进行有序的排列
```
from collections import *
z=OrderedDict([('AA',12),('CC',22),('BB',100)])
print(z)
```
## defaultdict
dict如果访问了不存在的key，会出异常。defaultdict可以给出一个默认值，如果访问的key不存在就返回这个默认值
```
zz=defaultdict(int)
zz['zzzz']=100
print(zz['zzzz'])
print(zz['zzzz1'])
```

## namedtuple
以通过namedtuple 为元组的每个索引设置名称，然后通过「属性名」来访问
```
Stu=namedtuple('Stu',['name','id','age'])
stutuple=Stu('lz',645,22)
print(stutuple.name)
print(stutuple.id)
```
## deque
双向队列，主要方法有
- pop
- popleft
- append
- appendleft
- extend 
- extendleft
- rotate(int)    旋转
     - rotate(2) 将尾部的两个元素移动到头部
     -rotate(-2) 将头部的两个元素移动到头尾部
 

























