---
layout:     post
title:      Spring in Action读书笔记-装配Bean
date:  2017-11-14
category:   Spring in Action读书笔记
tags:   [Java EE,Spring in Action读书笔记]
---
#### 自动扫描装配
`@component(id)`  注册为组件，也可以使用`@Named`  
id指定了组件的ID，默认不指定id的时候ID为类名首字母小写

```
使用Java代码开启自动扫描组件,扫描配置类所在的包下面的全部的类
@Configuration
@ComponentScan
class config{
}
xml方式:
<comtext:component-scan base-package=" ">
```
为Component设置扫描组件的基础包  
```
@ComponentScan(basePacages{"a","b"})
@ComponentScan(basePacageClasses{"C.class","d.class"}) c和d两个类所在的包的所有的类都会被扫描
```
` @Autowired ` 进行自动注入   等价于@inject，前者是Spring中特有的注解
```
Spring如果找不到bean进行注入的话，会抛出异常，可以把Autowired的required属性设置为false
```
### Java代码装配
`@Bean`注解会告诉Spring这个方法将会返回一个对象，该对象要注册为Spring应用上下文中的bean。方法体中包含了最终产生bean实例的逻辑。  
默认情况下，Spring中的bean都是单例的  
@Bean注解的方法可以采用任何必要的Java功能来产生bean实例。优点在于非常的灵活
```
@Configuration
public class Config {

    @Bean
    public CD getCD() {
        return new CD();
    }
    @Bean
    public CDplayer getplayer() {
        return new CDplayer(getCD());
    }
    @Bean
    public CDplayer getplayer2(CD cd) {
        return new CDplayer(cd);
    }
}
```
上面的两个装配CDplayer Bean的方法中getplayer2的处理更好，不需要CD的定义跟他在同一类中 。  
通过上面的方法就把CD注入进去了。

### XML装
`<bean/>` Spring 发现这个元素之后，会调用类构造方法来创建一个bean  
  
  构造器注入：
  - `<constructor-arg>`  
  - c-命名空间  
     - c:cd-ref="id"  ++其中cd是构造方法的参数名字
     - c:_0-ref="id"
      - c:_-ref="id"  
 c-命名空间更有优势，但是无法装配集合，装配集合应该使用`<construcror-arg>`
```
 <bean class="mediaPlayer.CD" id="cd"/>
 <bean class="mediaPlayer.CDplayer" c:_0-ref="cd"/>
 ```
属性注入：**属性注入的时候的属性要设置其setter方法**
- <property name="" ref=""或者value=""/>
- p-命名空间   p:cd-ref="id"
p命名空间也是不能够装配集合，但是可以通过util-命名空间达到相似的目的
```xml
<bean class="mediaPlayer.CD" id="cd" p:list-ref="dolist"/>
<bean class="mediaPlayer.CDplayer" c:_0-ref="cd"/>
<util:list id="dolist">
    <value>hello</value>
    <value>world!</value>
</util:list>
```
### 不同方式之间混合使用
1. Java Config中导入xml配置
- 在一个Java Config中导入另外一个Java Config  
```
@Configuration
@Import(AnotherConfig.class)
public class Config {}
```
- 导入xml则改为`@ImportResource("../../resources/spingcontext.xml")`
2. xml中导入Java Config配置
- xml之间使用`<import>`标签
```xml
<import resources="im.xml"/>
```
- xml导入java配置，只需要将配置类的bean装配进来就可以了，和装配普通的bean相同

























