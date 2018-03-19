---
layout:     post
title:     处理静态资源
date:      2018-03-19
category:   SpringMVC
tags:   [SpringMVC]
---

优雅REST风格的资源URL不希望带 .html 或 .do 等后缀.由于早期的Spring MVC不能很好地处理静态资源，所以在web.xml中配置DispatcherServlet的请求映射，往往使用 *.do 、 *.xhtml等方式。这就决定了请求URL必须是一个带后缀的URL，而无法采用真正的REST风格的URL。也就是说如果使用"/"进行拦截，Sping MVC会把静态资源的请求也捕获，由于Controller中没有静态资源对应的映射，就会出现异常。

在Spring 3.0中给出了两种处理方案：

### `<mvc:default-servlet-handler />`

在springMVC-servlet.xml中配置`<mvc:default-servlet-handler />`后，会在Spring MVC上下文中定义一个org.springframework.web.servlet.resource.DefaultServletHttpRequestHandler，它会像一个检查员，对进入DispatcherServlet的URL进行筛查，如果发现是静态资源的请求，就将该请求转由Web应用服务器默认的Servlet处理，如果不是静态资源的请求，才由DispatcherServlet继续处理。

一般Web应用服务器默认的Servlet名称是"default"，因此DefaultServletHttpRequestHandler可以找到它。如果你所有的Web应用服务器的默认Servlet名称不是"default"，则需要通过default-servlet-name属性显示指定：

`<mvc:default-servlet-handler default-servlet-name="所使用的Web服务器默认使用的Servlet名称" />`

### `<mvc:resources />`

`<mvc:resources />`更进一步，由Spring MVC框架自己处理静态资源，并添加一些有用的附加值功能。

首先，`<mvc:resources />`允许静态资源放在任何地方，如WEB-INF目录下、类路径下等，你甚至可以将JavaScript等静态文件打到JAR包中。通过location属性指定静态资源的位置，由于location属性是Resources类型，因此可以使用诸如"classpath:"等的资源前缀指定资源位置。传统Web容器的静态资源只能放在Web容器的根路径下，`<mvc:resources />`完全打破了这个限制。

其次，`<mvc:resources />`依据当前著名的Page Speed、YSlow等浏览器优化原则对静态资源提供优化。你可以通过cache-period属性指定静态资源在浏览器端的缓存时间，一般可将该时间设置为一年，以充分利用浏览器端的缓存。在输出静态资源时，会根据配置设置好响应报文头的Expires 和 Cache-Control值。

在接收到静态资源的获取请求时，会检查请求头的Last-Modified值，如果静态资源没有发生变化，则直接返回303相应状态码，提示客户端使用浏览器缓存的数据，而非将静态资源的内容输出到客户端，以充分节省带宽，提高程序性能。

```
<mvc:resources mapping="/resource/**" location="/" cache-period=""/>
拦截以resource开头的资源，通过location对应的根目录，找到**所指代的文件的位置。
```

### 多Mapping的执行顺序问题

在开启了上面两个静态处理(一般只选择一个)之后，需要开启`<mvc:annotation-driven/>`注解，启动dispatcherServlet的拦截功能。这时候相当于又三个位置都要对于进站资源进行拦截。

`DefaultAnnotationHandlerMapping`的order属性值是：0

`<mvc:resources/>`自动注册的 SimpleUrlHandlerMapping的order属性值是： 2147483646

`<mvc:default-servlet-handler/>`自动注册 的SimpleUrlHandlerMapping 的order属性值是： 2147483647

spring会先执行order值比较小的。当访问一个a.jpg图片文件时，先通过 DefaultAnnotationHandlerMapping 来找处理器，一定是找不到的，我们没有叫a.jpg的Controller。再按order值升序找。