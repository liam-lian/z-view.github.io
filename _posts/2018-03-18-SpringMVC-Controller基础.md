---
layout:     post
title:      Controller基础
date:      2018-03-18
category:   SpringMVC
tags:   [SpringMVC]
---

### @RequextMapping

```Java
@AliasFor("path")
String[] value() default {};

RequestMethod[] method() default {};

String[] params() default {};

String[] headers() default {};
```

通过上面的四个域可以精确的限定url要求

```
  params和headers支持简单的表达式：
– param1: 表示请求必须包含名为param1 的请求参数
– !param1: 表示请求不能包含名为param1 的请求参数
– param1!= value1: 表示请求包含名为param1 的请求参数，但其值不能为value1
– {“param1=value1”, “param2”}: 请求必须包含名为param1和param2的两个请求参数，且param1参数的值必须为value1
```

其中的url支持ant风格的通配符的注解

| 通配符 | 说明                    |
| ------ | ----------------------- |
| ?      | 匹配任何单字符          |
| *      | 匹配0或者任意数量的字符 |
| **     | 匹配0或者更多的目录     |

测试代码

```Java
@RequestMapping( value = "/a/b*/**/c",
				method = RequestMethod.GET,
				params = {"name", "id=3"},
				headers = {"myHeader"})
@ResponseBody
public String testParaAndHeader(){
    return "success";
}
```

### HiddenHttpMethodFilter

HiddenHttpMethodFilter捕获进入容器中的请求，然后根据请求中的_method参数所指定的值将请求包装为其他类型的请求。由于浏览器发出的只能是GET和POST请求，使用此过滤器可以将请求包装为DELETE或者PUT等请求，有利于使用RESTful风格的请求。需要在发送 POST 请求时携带一个 _method, 值为 DELETE 或 PUT，这样请求就会被包装成为相应的请求了。

```Xml
//web.xml中配置Filter
<filter>
    <filter-name>httpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>httpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>

```

```Java
//RESTful风格的请求
@ResponseBody
@RequestMapping(value = "/filter/{id}",method = RequestMethod.DELETE)
public String testHiddenHttpMethodFilterdelete(@PathVariable Integer id){
    return id+"==delete";
}
@ResponseBody
@RequestMapping(value = "/filter/{id}",method = RequestMethod.PUT)
public String testHiddenHttpMethodFilterput(@PathVariable Integer id){
    return id+"==put";
}
```

```Java
//源码
public class HiddenHttpMethodFilter extends OncePerRequestFilter {
   //指定包装请求类型的参数，默认为_method
   private String methodParam = DEFAULT_METHOD_PARAM;
   public void setMethodParam(String methodParam) {
      Assert.hasText(methodParam, "'methodParam' must not be empty");
      this.methodParam = methodParam;
   }
   @Override
   protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
         throws ServletException, IOException {
      String paramValue = request.getParameter(this.methodParam);
      if ("POST".equals(request.getMethod()) && StringUtils.hasLength(paramValue)) {
         String method = paramValue.toUpperCase(Locale.ENGLISH);
         HttpServletRequest wrapper = new HttpMethodRequestWrapper(request, method);
         filterChain.doFilter(wrapper, response);
      }
      else {
         filterChain.doFilter(request, response);
      }
   }
}
```

### @RequestParam和@RequestHeader

```Java
//参数有：required、defaultValue
@RequestMapping("/testRequestParam")
@ResponseBody
public String testRequestParam(@RequestParam(value = "id",required = false,defaultValue = "0") int id){
    return String.valueOf(id);
}
//@RequestHeader 与 @RequestParam用法相同
```

### @CookieValue

@CookieValue用来取得Cookie的值，用法实际上与@RequestParam相同

```Java
@ResponseBody
@RequestMapping("/testCookieValue")
public String testCookieValue(@CookieValue(value = "cookieID",required = false) String id , HttpServletResponse response){
    if(id==null){
        response.addCookie(new Cookie("cookieID","theCookieID"));
    }
    return id;
}
```

### 使用POJO作为入参

Spring可以直接根据POJO的参数名将对应的值填充到对应的参数中，**重要的是支持级联属性**

```Java
@RequestMapping("/testPOJO")
public void testPOJO(Person person){
    System.out.println(person);
}
注意请求时的参数为：localhost:8088/testPOJO?name=lz&id=123&address.city=asd&address.province=weqwe
其中的address.city就是直接给级联属性赋值。对于自动填充的POJO类一定要写setter和getter方法
```

### 对于原生Servlet API的支持
- HttpServletRequest
- HttpServletResponse
- HttpSession
- java.security.Principal
- Locale
- InputStream	 OutputStream
- Reader 	Writer


### 模型

模型中的数据被放入到请求域(request)中，传递给视图。**model模型数据的作用域与request相同**

##### ModelAndView  

不仅仅返回视图的名称同时可以带上Model的数据。(一般作为返回值)

##### Map和Model

一般作为参数，在函数中对于他们的操作处理，会被放到request请求域中，然后传递给View

```Java
@RequestMapping("/testModel")
public String testModel(Model model){
    model.addAttribute("lz","lzzz");
    return "forward:/zz";
}
@RequestMapping("/zz")
@ResponseBody
public String testModel1(Model model, HttpServletRequest request){
    return (String) request.getAttribute("lz");
}
```

上面的例子中，/testModel首先将("lz","lzzz")放到了Model中，实际上这个Model最终被放到了请求域中，经过forwawd之后，就可以直接从请求域中把"lz"取出来了。

###  @SessionAttribute

若希望在多个请求之间共用某个模型属性数据，则可以在控制器类上标注一个@SessionAttributes, Spring MVC
将在模型中对应的属性暂存到HttpSession 中。 @SessionAttributes 除了可以通过属性名指定需要放到会
话中的属性外，还可以通过模型属性的对象类型指定哪些模型属性需要放到会话中。**注意注解是放在类上面的**

```Java
@Controller
@SessionAttributes(value = "name",types = Address.class)
public class HomeController {

    @ResponseBody
    @RequestMapping("/testSessionAttribute")
    public String testSessionAttribute(HttpSession session,Model model){
        String result="";
        if(session.getAttribute("name")==null){
            model.addAttribute("name","lz"); //放入模型的同时也放入到Session
        }else {
            result= (String) session.getAttribute("name");
        }
        if(session.getAttribute("address")==null){
            model.addAttribute(new Address("BJ","BeiJing"));
        }else {
            result+=(Address)session.getAttribute("address");
        }
        return result;
    }
｝
```

在这个例子中，第一次访问/testSessionAttribute返回为空，后面访问的时候就能够通过Session得到内容了。。关键在于对于模型的操作不但将属性放到了RequestScope也放到了SessionScope。

### @ModelAttribute

@ModelAttribute 标记的方法, 会在每个目标方法执行之前被 SpringMVC 调用!

@ModelAttribute 注解也可以来修饰目标方法 POJO 类型的入参, 其 value 属性值有如下的作用:

 *  SpringMVC 会使用 value 属性值在 implicitModel 中查找对应的对象, 若存在则会直接传入到目标方法的入参中.
 *  SpringMVC 会以value 为 key, POJO 类型的对象为value, 存入到 request 中.


```Java
@ModelAttribute
public void getPerson(@RequestParam(value = "id" ,required = false)Integer id, Model model){
    if(id!=null){
        model.addAttribute(new Person("name",id));
    }
}

@RequestMapping("/ModelAttribute")
@ResponseBody
public String testModel(Person person){
    return person.toString();
}
```

执行localhost:8088/ModelAttribute?id=100，得到的结果是Person{name='name', id=100, address=null}
就是说在执行testModel方法之前首先执行了getPerson方法，将person对象已将放入到容器中去了，之后在入参处实际上获得的是之前得到的参数

注意: 在 @ModelAttribute 修饰的方法中, 放入到 Map 时的键需要和目标方法入参类型的第一个字母小写的字符串一致，因此默认都不进行修改最好了！否则，需要在目标方法入参的时候，使用@ModelAttribute进行名字的指定。

### SpringMVC 确定目标方法 POJO 类型入参的过程

1. 确定一个 key:
  1). 若目标方法的 POJO 类型的参数木有使用 @ModelAttribute 作为修饰, 则 key 为 POJO 类名第一个字母的小写
  2). 若使用了  @ModelAttribute 来修饰, 则 key 为 @ModelAttribute 注解的 value 属性值.
2. 在 implicitModel 中查找 key 对应的对象, 若存在, 则作为入参传入。【这里是request作用域的存在，一般情况下可能是@ModelAttribute 修饰的方法中进行的初始化，或者是forward之前进行的初始化】
3. 若 implicitModel 中不存在 key 对应的对象, 则检查当前的 Handler 是否使用 @SessionAttributes 注解修饰,  若使用了该注解, 且 @SessionAttributes 注解的 value 属性值中包含了 key, 则会从 HttpSession 中来获取 key 所对应的 value 值, 若存在则直接传入到目标方法的入参中. 若不存在则将     **抛出异常**. 【这里是Seesion作用域的存在，一般是在之前通过Session曾经放入进入HttpSession中，特别注意异常】
4. 若 Handler 没有标识 @SessionAttributes 注解或 @SessionAttributes 注解的 value 值中不包含 key, 则
  会通过反射来创建 POJO 类型的参数, 传入为目标方法的参数
5. SpringMVC 会把 key 和 POJO 类型的对象保存到 implicitModel 中, 进而会保存到 request 中.

```Java
@Controller
@SessionAttributes("person")
public class NewController {
		@RequestMapping("/addSession")
        public void addSession(Model model){
            model.addAttribute(new Person("lz",111));
        }

        @RequestMapping("/ModelAttribute")
        @ResponseBody
        public String testModel(Person person){
            return person.toString();
        }
｝
```

上面代码，首先addSession，将person加入到Session作用域中，之后ModelAttribute需要确定POJO参数，发现在implicitModel 没有，但是在Session作用域中存在，因此从Session中拿到了。注意如果不首先执行addSession会出异常，因为@SessionAttributes("person")告诉我们说person就在Session中，而/ModelAttribute没有找到就会出错

```Java
@RequestMapping("/addSession")
public String addSession(Model model){
    model.addAttribute(new Person("lz",111));
    return "redirect:/ModelAttribute";
}

@RequestMapping("/ModelAttribute")
@ResponseBody
public String testModel(Person person){
    return person.toString();
}
```

这个例子与上面相同，通过重定向操作理论上会丢失request中的属性数据，但是使用SessionAttributes通过Session的方式将数据保留了下来！
