---
layout:     post
title:      Exchanger
category:   JUC
tags:   [JUC]
---


Exchanger 可以看作是一种Barrier。仅仅试用于在两个线程之间交换数据。  
当一个线程调用exchang()方法的时候，会进入阻塞状态，直到另外一个线程也调用了exchange()方法为止。  

`public V exchange(V x)`exchange()方法传入一个参数，返回一个参数作为两个进程之间互相交换的数据【当然也可以交换空数据】