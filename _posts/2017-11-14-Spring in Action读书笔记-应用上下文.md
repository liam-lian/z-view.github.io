---
layout:     post
title:      Spring in Action读书笔记-应用上下文
date:  2017-11-14
category:   Spring in Action读书笔记
tags:   [Java EE,Spring in Action读书笔记]
---
SPring应用上下文就是Spring容器：  
- AnnotationConfigApplicationContext：从一个或多个基于Java的配置类中加载Spring应用上下文。
- AnnotationConfigWebApplicationContext：从一个或多个基于Java的配置类中加载Spring Web应用上下文。
- ClassPathXmlApplicationContext：从类路径下的一个或多个XML配置文件中加载上下文定义，把应用上下文的定义文件作为类资源。
- FileSystemXmlApplicationContext：从文件系统下的一个或多个XML配置文件中加载上下文定义。
- XmlWebApplicationContext：从Web应用下的一个或多个XML配置文件中加载上下文定义。
  
Bean的生命周期  
  
 **几乎每一个阶段都可以进行定制，这部分知识后续补充**
   
 

