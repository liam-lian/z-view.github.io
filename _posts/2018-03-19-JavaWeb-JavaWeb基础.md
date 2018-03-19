---
layout:     post
title:      JavaWeb基础
date:      2018-03-9
category:   JavaWeb
tags:   [JavaWeb]
---
## JavaWeb基础
### Cookie

cookie的内容主要包括：key，val，过期时间，路径、域。

路径与域一起构成cookie的作用范围。浏览器检查所有存储的cookie，如果某个cookie所声明的作用范围大于等于将要请求的资源所在的位置，则把该cookie附在请求资源的HTTP请求头上发送给服务器。

若不设置过期时间，则表示关闭浏览器窗口，这个cookie就消失。称为会话cookie，一般保存在内存中。

若设置了过期时间，浏览器就会把cookie保存到硬盘上，关闭后再次打开浏览器，这些cookie仍然有效直到超过设定的过期时间。[存储](http://cpro.baidu.com/cpro/ui/uijs.php?adclass=0&app_id=0&c=news&cf=1001&ch=0&di=128&fv=11&is_app=0&jk=f27d78b457762e6d&k=%B4%E6%B4%A2&k0=%B4%E6%B4%A2&kdi0=0&luki=6&n=10&p=baidu&q=65035100_cpr&rb=0&rs=1&seller_id=1&sid=6d2e7657b4787df2&ssp2=1&stid=0&t=tpclicked3_hc&td=1836545&tu=u1836545&u=http:%2F%2Fwww.bubuko.com%2Finfodetail-388671.html&urlid=0)在硬盘上的cookie可以在不同的浏览器进程间共享，比如两个IE窗口。而对于保存在[内存](http://cpro.baidu.com/cpro/ui/uijs.php?adclass=0&app_id=0&c=news&cf=1001&ch=0&di=128&fv=11&is_app=0&jk=f27d78b457762e6d&k=%C4%DA%B4%E6&k0=%C4%DA%B4%E6&kdi0=0&luki=5&n=10&p=baidu&q=65035100_cpr&rb=0&rs=1&seller_id=1&sid=6d2e7657b4787df2&ssp2=1&stid=0&t=tpclicked3_hc&td=1836545&tu=u1836545&u=http:%2F%2Fwww.bubuko.com%2Finfodetail-388671.html&urlid=0)里的cookie，不同的浏览器有不同的处理方式。

`cookie.setMaxAge(2000);`设置过期时间

路径值得的类似于`/testCookie`，域类似于`www.baidu.com`

Cookie的设置实在服务器的response头中定义的

```
Set-Cookie: name=xyz; expires=Friday, 04-Feb-07 22:03:38 GMT; path=/; domain=runoob.com
```

不能直接删除一个cookies，要想删除只能新创建一个同名 的cookie，然后设置maxAge=0：

```java
Cookie cookie=new Cookie（"K","V"）;
cookie.setMaxAge(0);
response.addCookie(cookie)
```

对于Web浏览器而言，每台Web服务器最多支持20个Cookie

###Session管理

Session的创建：调用HttpServletRequest的getSession方法创建。这个方法返回一个HttpSession对象，如果不存在Session，就创建一个返回。如果指定参数为false，则不会创建，而是返回null。

SessionId：在创建了Session的同时，服务器会为该Session生成唯一的Session id，而这个Session id在随后的请求中会被用来重新获得已经创建的Session；在Session被创建之后只会保存在服务器中，发到客户端的只有SessionId；当客户端再次发送请求的时候，会将这个SessionId带上，服务器接受到请求之后就会依据SessionId找到相应的Session，从而再次使用之。tomcat的ManagerBase类提供创建sessionid的方法：随机数+时间+jvmid

Session删除：1、超时 2、调用HttpSession.invalidate()  3、服务器关闭

服务器端的内存中，不过可以通过某些方式进行持久化

session不会因为浏览器的关闭而删除。但是**存有session ID的cookie**的默认过期时间是会话级别。也就是用户关闭了浏览器，那么存储在客户端的session ID便会丢失，但是存储在服务器端的session数据并不会被立即删除。从客户端即浏览器看来，好像session被删除了一样。

是指Session有效时间，超过有效时间没有访问这个Session，Session就会失效，如果访问了重新计时。
如果设置的值为零或负数，则表示会话将永远不会超时。

```
session.setMaxInactiveInterval(30 * 60);  //单位秒
<session-config> 
    <session-timeout>30</session-timeout> //单位分钟
 </session-config> 
```

`session.getId()`获得JSESSIONID的值



由于cookie可能会被禁用，可以使用以下的方式

1，URL重写

在网址后面增加参数，作为传递的依据.     格式是： 地址？k=v&k=V

2，使用隐藏的表单来传递数据  type='hidden'即可 //实际上跟url重写原理一致，不建议

3，上面的两种方法适合于不需要跨越很多页面的情况。使用cookie解决这些问题。

```
HttpSession session=req.getSession();
if(session.isNew()){
    System.out.println("新的Session");
    session.setAttribute("Time",System.currentTimeMillis());
}
System.out.println(session.getAttribute("Time"));
System.out.println(session.getId());
```

Java Web中提供了方法用于URl重写

```
 String HttpServletResponse.encodeRedirectURL(String);
 String response.encodeURL(String);
```



### Servlet接口

Servlet接口==>GenericServlet抽象类==>HttpServlet实现类
ServletRequest接口==>HttpServletRequest接口==>HttpServletRequestWrapper实现类
ServletResponse接口==>HttpServletResponse接口==>HttpServletResponseWrapper实现类

```
public void init(ServletConfig config) throws ServletException;
public void service(ServletRequest req, ServletResponse res)throws ServletException, 							IOException;
public void destroy();
public ServletConfig getServletConfig();
public String getServletInfo();
```

init方法会传入ServletConfig()，必须把ServletConfig传递给一个类变量，从中取得Servlet的初始化参数，例如在GenericServlet有如下定义:

```
private transient ServletConfig config;//线程安全
public void init(ServletConfig config) throws ServletException {
	this.config = config;
	this.init();}
```

### ServletContext

ServletContext表示Servlet应用程序，每个Web应用只有一个上下文，但是如果实在分布式环境下，每一台jvm有一个ServletContext对象。通过操作Attribute可以进行参数的处理。

### ServletConfig

主要作用在于获取Servelet的初始化参数，还可以获得ServletContext对象
`servletConfig.getServletContext`

### 文件上传

文件上传主要用到的是@MultiPartConfig以及Part接口

```Java
@WebServlet("/file")
@MultipartConfig
public class FileUpload extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Collection<Part> parts = req.getParts();
        for (Part part : parts){
            String filename=part.getSubmittedFileName();
            String path=getServletContext().getRealPath("/WEB-INF")+"/"+filename;
            part.write(path);
            System.out.println(path);
            resp.setContentType("text/plain");
            resp.setCharacterEncoding("utf-8");
            resp.getWriter().print("收到了文件"+filename);
        }
    }
}
```

Part接口可以获得文件的内容并且决定文件的创建和删除等工作。