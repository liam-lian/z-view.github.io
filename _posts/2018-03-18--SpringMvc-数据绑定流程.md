---
layout:     post
title:      SpringMvc的数据绑定流程
date:      2018-03-18
category:   SpringMVC
tags:   [SpringMVC]
---

# [SpringMvc的数据绑定流程](http://www.cnblogs.com/hujingwei/p/5349296.html)

在SpringMvc中会将来自web页面的请求和响应数据与controller中对应的处理方法的入参进行绑定，即数据绑定。流程如下：
- 1.SpringMvc主框架将ServletRequest对象及目标方法的入参实例传递给WebDataBinderFactory实例，以创建DataBinder实例对象

- 2.DataBinder对象调用装配在SpringMvc上下文中的ConversionService组件进行数据类型转换，数据格式化工作，将Servlet中的请求信息填充到入参对象中。

- 3.调用Validator组件对已经绑定了请求消息的入参对象进行数据合法性校验，并最终生成数据绑定结果BindingData对象

- 4.SpringMvc抽取BindingResult中的入参对象和校验错误对象，将它们赋给处理方法的相应入参。

  总结起来：大致流程是  绑定（bindingResult）-->  数据转换（conversionService）--> 校验（validators）

**数据转换**

​    SpringMvc上下文中内建了很多转换器，可以完成大多数Java类型转换工作。但是如果就要转换成我们自定义的类型时，那么我们就要自定义类型转换器，并将其加入到conversionService中去，conversionService中包含了很多SpringMvc内建的转换器。

​    ConversionService是SpringMvc类型转换体系的核心接口，可以利用ConversionServiceFactoryBean在Spring的IOC容器中定义一个ConversionService，Spring将自动识别出IOC容器中的ConversionService，并在bean属性配置及SpringMvc处理方法入参绑定等场合使用它进行数据转换。

在SpringMvc的配置文件中配置转换器：

　Spring支持三种类型的转换器接口，实现任意一个转换器接口都可以作为自定义转换器注册ConversionServiceFactoryBean中：

- Converter<S,T>:将S类型的对象转为T类型对象

- ConverterFactory:将相同系列多个"同质"Converter封装在仪器，如果希望将一种类型的对象转换为一种类型及其子类的对象（例如将String转化为Number及Number的子类）

- GenericConverter:会根据源类对象与目标类对象所在的宿主类中的上下文信息进行类型转换。`<mvc:annotation-driven conversion-service="conversionService"/>`会将自定义的ConversionService注册到SpringMvc的上下文中去。

  给出一个自定义类型转换器的实例

  ```
  @RequestMapping("/add")
  @ResponseBody
  public void ttt(@RequestParam("adds") Address address){
      System.out.println(address);
  }
  ```

  ```
  @Component
  public class MyConverter implements Converter<String, Address> {


      public Address convert(String source) {
          if(source!=null){
              String[] split=source.split("-");
              if(split.length<2) return null;
              return new Address(split[0],split[1]);
          }
          return null;
      }
  }
  ```

  ```
  <mvc:annotation-driven conversion-service="converts"/>
  <bean id="converts" class="org.springframework.context.support.ConversionServiceFactoryBean">
      <property name="converters">
          <set>
              <ref bean="myConverter"/>
          </set>
      </property>
  </bean>
  ```

## 关于mvc:annotation-driven

​     <mvc:annotation-driven/>会自动注册ReuqestMappingHandlerMapping、ReuqestMappingHandlerHandler、ExceptionHanderExceptionResolver三个bean。还提供以下支持：

​     -支持使用ConversionService实例对表单参数进行类型转换

​    -支持使用@NumberFormat annotation @DateTimeFormat 注解完成数据类型的格式化

​    -支持使用@Valid注解对JavaBean实例进行JSR303验证

​    -支持使用@RequestBody 和@ResponseBody注解

　