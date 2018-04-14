---
layout:     post
title:      AOP详解
date:      2018-03-11
category:   Spring
tags:   [Spring]
---
## Spring AOP

- @AfterReturning有returning属性，可以获得返回值，只有方法执行成功并且返回之后才会执行
- @AfterThrowing有throwing属性，可以获得异常的值
- @EnableAspectJAutoProxy启动AOP的功能
- @Order标明切面的优先级，其中value的值越小，切面越先被执行

## JoinPoint 对象

JoinPoint对象封装了SpringAop中切面方法的信息,在切面方法中添加JoinPoint参数,就可以获取到封装了该方法信息的JoinPoint对象. **常用api:**

| 方法名                    | 功能                                                         |
| ------------------------- | ------------------------------------------------------------ |
| Signature getSignature(); | 获取封装了署名信息的对象,在该对象中可以获取到目标方法名,所属类的Class等信息 |
| Object[] getArgs();       | 获取传入目标方法的参数对象                                   |
| Object getTarget();       | 获取被代理的对象                                             |
| Object getThis();         | 获取代理对象                                                 |

## ProceedingJoinPoint对象

ProceedingJoinPoint对象是JoinPoint的子接口,该对象只用在@Around的切面方法中, 添加了

 `Object proceed() throws Throwable //执行目标方法` 

`Object proceed(Object[] var1) throws Throwable //传入的新的参数去执行目标方法` 两个方法.

```Java
	@Component
	@Aspect
	@EnableAspectJAutoProxy
	@Order(1)
	public class AspectConfig {

	    @Pointcut("execution(* VideoLearn.aspect.*.*(..))")
	    public void pointCut() {
	    }

	    @Before("pointCut()")
	    public void beforeCal() {
		System.out.println("前置通知...");
	    }

	    @After("pointCut()")
	    public void afterCal(JoinPoint joinPoint) {
		System.out.println("后置通知...");
		    System.out.println("目标方法名为:" + joinPoint.getSignature().getName());
		System.out.println("目标方法所属类的简单类名:" + joinPoint.getSignature().getDeclaringType().getSimpleName());
		System.out.println("目标方法所属类的类名:" + joinPoint.getSignature().getDeclaringTypeName());
		System.out.println("目标方法声明类型:" + Modifier.toString(joinPoint.getSignature().getModifiers()));
		//获取传入目标方法的参数
		Object[] args = joinPoint.getArgs();
		for (int i = 0; i < args.length; i++) {
		    System.out.println("第" + (i+1) + "个参数为:" + args[i]);
		}
		System.out.println("被代理的对象:" + joinPoint.getTarget());
		System.out.println("代理对象自己:" + joinPoint.getThis());
	    }

	    @AfterReturning(value = "pointCut()", returning = "returnvalue")
	    public void afterReturning(Object returnvalue) {
		System.out.println("返回通知,返回值为" + returnvalue);
	    }

	    @AfterThrowing(value = "pointCut()", throwing = "e")
	    public void afterThrowing(JoinPoint jp, Exception e) {
		System.out.println(jp.getSignature().getName() + " " + Arrays.asList(jp.getArgs()));
		System.out.println(e);
	    }

	    @Around(value = "pointCut()")
	    public Object around(ProceedingJoinPoint jp) {
		Object result = null;
		try {
		    System.out.println(">>Before通知的位置");
		    result = jp.proceed(new Object[]{100,3});
		    System.out.println(">>After通知的位置");
		} catch (Throwable throwable) {
		    System.out.println(">>AfterThrowing通知的位置"+throwable);
		    return 0;
		}
		System.out.println(">>AfterReturning通知的位置");
		return result;
	    }
	}
```

