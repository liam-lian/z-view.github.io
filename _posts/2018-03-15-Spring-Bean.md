---
layout:     post
title:      Bean
date:  2018-03-15
category:   Spring
tags:   [Spring]
---

Bean的生命周期

---

- 通过构造器或工厂方法创建 Bean实例
- – 为 Bean 的属性设置值和对其他Bean的引用
- 调用 Bean 的init方法
- Bean 可以使用了
- 当容器关闭时, 调用Bean的destroy方法

通过实现BeanPostProcessor接口，在Bean初始化前后会调用相应的BeforeInitilization和AfterInitilization方法，对于Bean进行管理

```
@Configuration
public class SpringCycleConfig {
    @Bean(initMethod = "init",destroyMethod = "des")
    public Car car(){
        return new Car("Aodi",100L);
    }
    @Bean
    public BeanPostProcessor beanPostProcessor(){  //注意前置处理器也要声明为Bean
        return new MyBeanPostProcessor();
    }
}
public class MyBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("Before Init..."+beanName);
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("After Init..."+beanName);

        if(bean.getClass().isInstance(Car.class)){
            ((Car)bean).setBrand("haha");
        }
        return bean;
    }
}
```

需要注意的是BeanPostProcessor是在每一个Bean(除了自身)的Init方法前后都会调用相关方法的，可以通过class进行区别处理

通过工厂方法配置Bean

---

意思是说对应的Bean是通过工厂模式获取的

- 静态工厂

  ```
  <!--静态工厂，直接调用工厂的静态的factory-method产生Bean-->
  <bean id="car" class="VideoLearn.Serve.StaticFactory" factory-method="getCar" c:_0="AoDi"/>
  ```

- 实例工厂

  ```
  <!--实例工厂，工厂需要新创建之后才能使用-->
  <bean id="dynamicFactory" class="VideoLearn.Serve.DynamicFactory"/>
  <bean id="carDynamic" factory-bean="dynamicFactory" factory-method="getCar" c:_0="Aodi"/>
  ```

使用FactoryBean来产生Bean

---

需要自定义类实现FactoryBean接口。

```
<bean id="car3" class="VideoLearn.Serve.MyfactoryBean" p:branch="Aodi1"/>
//需要实现三个方法
public class MyfactoryBean implements FactoryBean<Car> {

    private String branch;

    public void setBranch(String branch) {
        this.branch = branch;
    }

    @Override
    public Car getObject() throws Exception {
        return new Car(branch,100);
    }

    @Override
    public Class<?> getObjectType() {
        return Car.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

事实上，FactoryBean的方式与上面的使用工厂配置Bean的方法类似，只不过是通过接口直接告诉了容器我要使用的是工厂方法进行创建！



