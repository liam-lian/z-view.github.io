---
layout:     post
title:      Filter
date:      2018-03-16
category:   JavaWeb
tags:   [JavaWeb]
---
# [web应用中的Filter过滤器之基础概述](http://www.cnblogs.com/zwbg/p/5948111.html)

**1 过滤器概述**

　　当web容器接收到对一个资源的请求时，它将判断是否有过滤器与这个资源相关联，如果有，那么容器将把这个请求交给过滤器进行处理。在过滤器中，你可以改变请求的内容或者重新设置请求的报头信息，然后再将请求发送给目标资源。当目标资源对请求作出响应时，容器同样会将响应先转发给过滤器，在过滤器中，你可以对响应的内容进行转换，然后再将响应发送给客户端。从此过程中可以看出，客户端和目标资源并不需要知道过滤器的存在，也就是说，在web应用中部署过滤器，对客户端和目标资源是透明的。

　　在web应用中，你可以部署多个过滤器，这些过滤器组成过滤器链，过滤器链中的每一个过滤器负责特定的操作和任务，客户端的请求在这些过滤器之间传递，直到目标资源。

　　在请求资源时，过滤器链中的过滤器将依次对请求进行处理，并将请求传递给下一个过滤器，直到目标资源，在发送响应时，则按照相反的顺序对响应进行处理，直到客户端。

　　如图所示，由多个过滤器组成的过滤器链。

　　![img](https://images2015.cnblogs.com/blog/994276/201610/994276-20161011084158696-490545481.png)

　　过滤器并不是必须要将请求传递给下一个过滤器（或者目标资源），它也可以自行处理请求，然后发送响应给客户端，或者将请求发送给另外一个目标资源。

　　过滤器在web开发的一些主要应用：

　　1）对用户请求进行统一认证

　　2）对用户的访问请求进行记录和审核

　　3）对用户发送的数据进行过滤或者替换

　　4）对响应内容进行压缩，减少传输容量

　　5）对请求和响应进行加解密处理

　　6）触发资源访问事件

------

 

**2 Filter API**

与过滤器开发相关的接口和类都包含在javax.servlet或者javax.servlet.http包中，主要有：

javax.servlet.Filter接口

javax.servlet.FilterConfig接口

javax.servlet.FilterChain接口

**1)Filter接口**

开发过滤器要实现该接口，并提供一个公共的不带参数的构造方法。

该接口的方法：

`public abstract void init(FilterConfig filterconfig)throws ServletException`
　　web容器调用该方法初始化过滤器，容器在调用该方法时，向过滤器传递FiterConfig对象，FiterConfig对象的用法和ServletConfig类似。利用FilterConfig对象可以得到ServletContext对象，以及在配置文件配置的过滤器的初始化参数，这个方法可以抛出ServletException异常，通知容器过滤器不能正常工作。

`public abstract void doFilter(ServletRequest servletrequest, ServletResponse servletresponse, FilterChain filterchain)throws IOException, ServletException`
　　doFilter()方法类似于Servlet中service()方法。当客户端请求目标资源时，容器就会调用与这个目标资源相关联的过滤器的doFilter()方法。在这个方法里，可以对请求和响应进行处理。在实现特定的操作后，可以调用chain.doFilter(request,response)将请求传给下一个过滤器，也可以直接向客户端返回响应，或者利用RequestDispatcher的forward()和include()方法，以及HttpServletResponse的sendRedirect()方法将请求**转向其他资源**。这个方法的请求和响应参数是ServletRequest ServletResponse ，也就是说，过滤器的使用并不依赖于具体的协议。

`public abstract void **destroy**()`

　　web容器调用该方法表示过滤器的生命周期结束，在这个方法中，可以释放过滤器使用的资源，

**与Servlet不同的是，Filter接口并没有相应的实现类可供继承，要开发过滤器，只能实现Filter接口。**

**2）FilterConfig接口**

类似于ServletConfig接口，用于在过滤器初始化时向其传递信息。FilterConfig接口由容器实现，容器将其实例作为参数传入过滤器对象的init()方法中。

`public abstract String getFilterName()`  以字符串形式返回在web.xml中配置的Filter的名字

`public abstract ServletContext getServletContext()`返回Servlet上下文对象的引用

**3）FilterChain接口**

该接口由容器实现，容器将其作为参数传入过滤器对象的doFilter()方法中。过滤器对象使用FiterChain对象调用过滤器链中的下一个过滤器。**如果此过滤器是链中的最后一个过滤器，那么将调用目标资源**。FilterChain接口只有一个方法：

`public abstract void doFilter(ServletRequest servletrequest, ServletResponse servletresponse)`

------

**3 过滤器在web.xml中的配置**

在实现一个过滤器后，需要在web.xml进行配置，这是通过`<filter>`和`<filter-mapping>`元素来完成的

注意，**filter的顺序是由其在web.xml文件中定义的顺序决定的**，如果使用注解@WebFilter进行定义则不可以制定顺序

`<filter>`元素用于在web应用程序中声明一个过滤器。

`<description> <display-name> <icon>`元素的作用是相同的。

`<init-param>`元素用于为过滤器指定初始化参数`

 Servlet容器对在web.xml中配置的每一个过滤器，只创建一个实例。与Servlet相似，容器将在同一个过滤器实例上运行多个线程来同时为多个请求服务，因此开发过滤器时，也应该注意线程安全的问题。如果在web.xml中，对同一个过滤器类声明了两次，那么容器会创建两个相同的过滤器类的实例。

`<filter-mapping>`元素用于指定过滤器关联的URL或者Servlet。

**`<filter-mapping>`元素结构解析：**

其中`<filter-name>`子元素的值必须是在`<filter>`元素中声明过的过滤器的名字。

`<url-pattern>`元素和`<servlet-name>`元素可以选择一个，`<url-pattern>`元素指定过滤关联的URL样式，`<servlet-name>`元素指定过滤器对应的Servlet。用户在访问`<url-pattern>`元素指定的URL上的资源或者`<servlet-name>`元素指定的Servlet时，该过滤器才会被容器调用。

`<filter-mapping>`还可以包含0到4个`<dispatcher>`元素，`<dispatcher>`元素指定过滤器对应的请求方式，可以是REQUEST INCLUDE FORWARD ERROR四种之一或者多种，默认是REQUEST。

**REQUEST**：当用户直接访问页面时，web容器将会调用过滤器。如果目标资源是通过RequestDispatcher的include()或者forward()方法访问时，那么该过滤器将不会被调用。

**INCLUDE**：如果目标资源是通过RequestDispatcher的include()方法访问时，那么该过滤器将被调用，除此之外，过滤器不会被调用。

**FORWARD**：如果目标资源时通过ResquestDispatcher的forward()方法访问时，那么该过滤器将会被调用，除此之外，过滤器不会被调用。

ERROR：如果目标资源是通过声明式异常处理机制调用时，那么该过滤器将被调用。除此之外，过滤器不会被调用。

示例代码

```
import java.io.IOException;  
  
import javax.servlet.*;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  
  
// 主要目的：过滤字符编码；其次，做一些应用逻辑判断等.  
// Filter跟web应用一起启动  
// 当web应用重新启动或销毁时，Filter也被销毁  
public class MyCharsetFilter implements Filter  
{  
    private FilterConfig config = null;  
      
    public void destroy()  
    {  
        System.out.println("MyCharsetFilter准备销毁...");  
    }  
  
    public void doFilter(ServletRequest arg0, ServletResponse arg1,FilterChain chain) throws IOException, ServletException  
    {  
        // 强制类型转换  
        HttpServletRequest request = (HttpServletRequest) arg0;  
        HttpServletResponse response = (HttpServletResponse) arg1;  
        // 获取web.xm设置的编码集，设置到Request、Response中  
        request.setCharacterEncoding(config.getInitParameter("charset"));  
        response.setContentType(config.getInitParameter("contentType"));  
        response.setCharacterEncoding(config.getInitParameter("charset"));  
        // 将请求转发到目的地  
        chain.doFilter(request, response);  
    }  
  
    public void init(FilterConfig filterconfig) throws ServletException  
    {  
        this.config = filterconfig;  
        System.out.println("MyCharsetFilter初始化...");  
    }  
}  
```

```
<filter>  
      <filter-name>filter</filter-name>  
      <filter-class>dc.gz.filters.MyCharsetFilter</filter-class>  
      <init-param>  
          <param-name>charset</param-name>  
          <param-value>UTF-8</param-value>  
      </init-param>  
      <init-param>  
          <param-name>contentType</param-name>  
          <param-value>text/html;charset=UTF-8</param-value>  
      </init-param>  
  </filter>  
  <filter-mapping>  
      <filter-name>filter</filter-name>  
      <url-pattern>/*</url-pattern>  
  </filter-mapping>  
```