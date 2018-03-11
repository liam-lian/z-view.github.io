---
layout:     post
title:      system.gc()和system.runFinalization()
date:  2017-12-06
category:   Java
tags:   [Java]
---
System.gc(); //告诉垃圾收集器打算进行垃圾收集，而垃圾收集器进不进行收集是不确定的 

System.runFinalization(); //强制调用已经失去引用的对象的finalize方法 

java中的finalize()方法
当垃圾收集器认为没有指向对象实例的引用时，会在销毁该对象之前调用finalize()方法。该方法最常见的作用是确保释放实例占用的全部资源。java并不保证定时为对象实例调用该方法，甚至不保证方法会被调用，所以该方法不应该用于正常内存处理。
