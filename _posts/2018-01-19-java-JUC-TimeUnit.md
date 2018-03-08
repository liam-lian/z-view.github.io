---
layout:     post
title:      TimeUnit.md
category:   JUC
tags:   [JAVA, JUC]
---
## TimeUnit
枚举类型
- MICROSECONDS  
- MILLISECONDS     
- NANOSECONDS
- SECONDS          
- MINUTES     
- HOURS      
- DAYS      
方法:
- convert(long duration, TimeUnit unit)的意思将duration这个时间转化为本对象（this）所表示的时间形式。
-  void  sleep(long timeout)
          使用此单元执行 Thread.sleep.这是将时间参数转换为 Thread.sleep 方法所需格式的便捷方法。  
          这样的可读性更强