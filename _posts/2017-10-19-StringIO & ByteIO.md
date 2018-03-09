---
layout:     post
title:      IO-StringIO & ByteIO
date:  2017-10-19
keywords:   IO
category:   java
tags:   [python,IO]
---
这两个类都是跟文件读写操作类似的，属于file-like Object类型  
常常用来作为临时的缓冲    
  
StringIO 和BytesIO 是在内存中操作str 和bytes 的方法，使得和读写文
件具有一致的接口。
```
from io import StringIO
f = StringIO()
f.write('hello')
f.write(' ')
f.write('world!') 
print(f.getvalue())
hello world!
```
---
```
from io import BytesIO
f = BytesIO()
f.write('中文'.encode('utf-8'))
print(f.getvalue())
b'\xe4\xb8\xad\xe6\x96\x87'
```
