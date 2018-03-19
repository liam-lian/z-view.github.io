---
layout:     post
title:      ServletListener与观察者模式
date:      2018-03-20
category:   JavaWeb
tags:   [JavaWeb]
---

## 观察者模式

　观察者模式又称为发布/订阅(**Publish/Subscribe**)模式，一个被观察的对象可以被多个观察者订阅，观察者在被观察对象中进行订阅之后，被观察对象如果自身状态发生了变化，回去迭代每一个观察者向他们发出通知(publish)，事实上是通过被观察的对象通知观察者的方式实现的。

Java中默认给出了观察者对象的实现接口，事实上自己编写一个观察者的实现也很简单。实现类

- Observable类

  被观察者继承该类即可。该类中有Observer的集合，当发生通知的时候轮询这个集合即可。在子类中需要给出观察者通知的位置调用notifyObservers()方法即可，该方法遍历Observer，执行每一个观察者的update方法。注意到，只有changed为true才会发出通知，因此是发生变化的观察点上setChanged，在想要通知的时候才正式notifyObservers()发出通知。

  ```Java
  public void notifyObservers(Object arg) {
      Object[] arrLocal;

      synchronized (this) {
          if (!changed)  return;
          arrLocal = obs.toArray();
          clearChanged();
      }

      for (int i = arrLocal.length-1; i>=0; i--)
          ((Observer)arrLocal[i]).update(this, arg);
  }
  ```

- Observer接口，只有update方法

  被观察者通过update通知观察者。

例子：

```Java
public class Obsered extends Observable {

    public void action(String act){
        setChanged();
        System.out.println("Obsered发生了变化");
        notifyObservers(act);
    }

    public static void main(String[] args) {
        Obsered obsered=new Obsered();
        obsered.addObserver(new ObserverImpl("观察者1"));
        obsered.addObserver(new ObserverImpl("观察者2"));
        obsered.action("通知1");
        obsered.addObserver(new ObserverImpl("观察者3"));
        obsered.action("通知2");
    }
}
```

```Java
public class ObserverImpl implements Observer{

    private String ObserverID;

    public ObserverImpl(String observerID) {
        ObserverID = observerID;
    }

    @Override
    public void update(Observable o, Object arg) {
        System.out.println("observer:"+ObserverID+"收到通知"+arg);
    }
}
```

### Java Web监听器

JavaWeb的监听器模式的实现原理与观察者模式类似，也是通过被观察者通知的方式进行的。

Java Web一共提供了八个监听器供使用，分别用于监听Request、Session和ServletContext等的创建与销毁、属性变化。Listener需要在web.xml中注册

```XML
<listener>
	<listener-class>com.we.mylistener</listener-class>
</listener>
```

#### ServletContext监听器

- ServletContextListener

ServletContextListener用于监听ServletContext的创建与销毁。服务器启动或者部署Web应用程序时执行contextInitialized()方法，服务器关闭或关闭Web应用程序时执行contextDestroyed()方法。该Listener可用于启动时获取web.xml文件中配置的初始化参数。

- ServletContextAttributeListener

ServletContextAttributeListener用于监听ServletContext对象添加、更新和移除属性。ServletContext添加属性时执行contextAdded()方法，ServletContext对象更新属性时执行contextReplaced()方法，Session对象移除属性时执行contextRemoved()方法。

#### ServletRequest监听器

- ServletRequestListener

ServletRequestListener用于监听Request的创建与销毁。用户每次请求Request都会执行requestInitialized()方法，Request处理完毕自动销毁前执行requestDestroyed()方法。需要注意的是如果一个页面内含有多个图片等元素，则请求一次页面可能会触发多次Request事件。

- ServletRequestAttributeListener

ServletRequestAttributeListener用于监听Request对象添加、更新和移除属性。Request添加属性时执行requestAdded()方法，Request对象更新属性时执行requestReplaced()方法，Request对象移除属性时执行requestRemoved()方法。

- AsycListener

### Session监听器

- HttpSessionListener

HttpSessionListener用于监听Session的创建与销毁。创建Session时执行sessionCreated()方法，销毁Session或Session超时时执行sessionDestroyed()方法。该Listener可用于收集在线者信息。

- HttpSessionAttributeListener

HttpSessionAttributeListener用于监听Session对象添加、更新和移除属性。Session添加属性时执行sessionAdded()方法，Session对象更新属性时执行sessionReplaced()方法，Session对象移除属性时执行sessionRemoved()方法。

- HttpSessionActivationListener

  为了节省内存，Servlet容器会对Session属性进行迁移或者序列化，一般，当内存比较低的时候，相对访问比较少的对象可以序列化到备用的存储设备中。HttpSessionActivationListener包含两个方法

  - sessionDidActive(HttpSessionEvent event)  当Session被激活(锐化时调用)
  - sessionWillPassivate(HttpSessionEvent event) 当Session被钝化时调用

- HttpSessionBindingListener

  - JavaBean感知监听

  HttpSessionBindingListener用于当JavaBean对象被放到Session里时执行valueBound()方法，当JavaBean对象从Session移除时执行valueUnBound()方法。JavaBean必须实现该Listener接口。

  - 创建一个JavaBean，实现HttpSessionBindingListener接口。

  ```Java
  public class User implements HttpSessionBindingListener {
      private String name;
      public void valueBound(HttpSessionBindingEvent event) {
          System.out.println("该JavaBean被添加到session中......");
      }
      public void valueUnbound(HttpSessionBindingEvent event) {
          System.out.println("该JavaBean从session中被移除......");
      }
  }
  ```

  创建一个Servlet用于将JavaBean对象放置到Session和从Session移除。

  ```Java
  public class BeanServlet extends HttpServlet {
      public void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
          HttpSession session = request.getSession();
          session.setAttribute("user", new User("zhangwuji", 18, "jiaozhu"));
      }
      public void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
          HttpSession session = request.getSession();
          session.removeAttribute("user");
      }
  }
  ```

  使用HttpSessionBindingListener监听器时，不需要配置web.xml文件。

#### 实例——统计当前在线人数(Session数量统计)

```Java
public class OnlineListener implements HttpSessionListener , ServletContextListener {
    @Override
    public void sessionCreated(HttpSessionEvent se) {
        ServletContext context=se.getSession().getServletContext();
        Integer count= (Integer) context.getAttribute("online");
        context.setAttribute("online",++count);
        System.out.println(se.getSession().getId()+"上线"+"在线人数:"+count);
    }

    @Override
    public void sessionDestroyed(HttpSessionEvent se) {
        ServletContext context=se.getSession().getServletContext();
        Integer count= (Integer) context.getAttribute("online");
        context.setAttribute("online",--count);
        System.out.println(se.getSession().getId()+"下线"+"在线人数:"+count);

    }

    @Override
    public void contextInitialized(ServletContextEvent sce) {
        sce.getServletContext().setAttribute("online",0);
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {

    }
}
```

