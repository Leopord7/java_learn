# Spring AOP 框架

## 第七章 一起来看AOP

### 含义

传统的OOP等方法无法很好的对业务中的系统需求（横切关注点）进行实现。

AOP：以Aspect对系统需求进行模块化封装。

织入：AO组件集成到OOP组件的过程。

### AOP历史

#### 静态AOP

最初的AspectJ，将横切点实现之后，通过特定的编译器以字节码的形式编译到各个模块中

- 优点：不会造成性能损失
- 缺点：灵活性不够，改变织入点位置需要修改AspectJ定义文件

#### 动态AOP

- **动态代理**：模块类要实现相应接口；使用反射性能稍逊
- **动态字节码增强**：生成子类将横切逻辑织入子类，若为final则失败
- java代码生成：EJB
- 自定义类加载器
- AOL扩展

### AOP术语

JoinPoint：连接点，可作为横切逻辑的织入点

PointCut：切入点，JoinPoint的描述方式。

Advice：通知，将被织入的横切逻辑

- Before Advice

- After Advice

    - After Advice
    - After returning Advice
    - After throwing Advice

    - Around Advice ：前后都执行横切逻辑

- Introduction

Aspect：切面，封装横切逻辑的概念实体（切入点+具体逻辑）

Weaver：织入器

- AspectJ：ajc
- JBoss：类加载器
- Spring：一组类

Target：目标对象

![image-20201022151148196](D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第三部分_Spring AOP 框架.assets\image-20201022151148196.png)

## 第八章 Spring AOP概述及其实现机制

### 概述

采用Java作为AOL满足80%需求，也集成了AspectJ，满足额外需求

### 实现机制

#### JDK动态代理

```java
public class RequestCtrlInvocationHandler implements InvocationHandler {
 	private static final Log logger = LogFactory.getLog(RequestCtrlInvocationHandler.class);

	private Object target;

 	public RequestCtrlInvocationHandler(Object target) {
 		this.target = target;
 	}

 	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
 		if(method.getName().equals("request"))
 		{
 			TimeOfDay startTime = new TimeOfDay(0,0,0);
 			TimeOfDay endTime = new TimeOfDay(5,59,59);
 			TimeOfDay currentTime = new TimeOfDay();
 			if(currentTime.isAfter(startTime) && currentTime.isBefore(endTime))
 			{
 				logger.warn("service is not available now.");
 				return null;
 			}
 			return method.invoke(target, args);
 		}
 		return null;
 	}
}
ISubject subject = 
    (ISubject)Proxy.newProxyInstance(ProxyRunner.class.getClassLoader(),new Class[]{ISubject.class} ,newRequestCtrlInvocationHandler(new SubjectImpl()));subject.request(); 
```

代理类也会实现相应接口，使用了反射，实际处理交给了InvocationHandler

具体原理见[博文](https://www.cnblogs.com/whirly/p/10154887.html)

#### CGLIB动态字节码生成



## 第九章 Spring AOP一世

### 时间

Spring刚发布至2.0之前

### JoinPoint

Spring中仅提供对方法执行的JointPoint

- 可以满足80%的需求，不足用AspectJ补足

### PointCut

Spring中以接口定义org.springframework.aop.Pointcut作为其AOP框架中所有Pointcut的最顶层抽象

```java
public interface Pointcut {
 	ClassFilter getClassFilter();			//对类进行匹配
 	MethodMatcher getMethodMatcher();		//匹配方法
 	Pointcut TRUE = TruePointcut.INSTANCE;	//一个简单实现 匹配所有对象上所有JoinPoint
} 
```

```java
public interface ClassFilter {
 	boolean matches(Class clazz);			
 	ClassFilter TRUE = TrueClassFilter.INSTANCE;	//返回该实例则匹配所有类
} 
```

```java
public interface MethodMatcher {
 	boolean matches(Method method, Class targetClass);
 	boolean isRuntime();
 	boolean matches(Method method, Class targetClass, Object[] args);
 	MethodMatcher TRUE = TrueMethodMatcher.INSTANCE;
} 
```

#### MethodMatcher

- 通过isRuntime()决定是否匹配参数
- 不匹配为Static型，性能更好
- 匹配为Dynamic型，不能缓存尽量避免

#### 常见的PointCut

- NameMatchMethodPointcut：匹配名称，可模糊匹配
- AnnotationMatchingPointcut：基于特定注解进行匹配
- ......

可以自定义PointCut，可以作为Bean注入到IOC容器中

### Advice

#### 背景

加入了AOP Alliance组织，旨在规范AOP使用

#### 关系

![image-20201022173156222](D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第三部分_Spring AOP 框架.assets\image-20201022173156222.png)

#### per-class per-instance

- 前者表示该Advice在目标类中所有实例共享，除了introduction都是此类

    - before Advice

        ```java
        public class ResourceSetupBeforeAdvice implements MethodBeforeAdvice {
             private Resource resource;
             public ResourceSetupBeforeAdvice(Resource resource)
             {
             	this.resource = resource;
             }
             public void before(Method method, Object[] args, Object target) throws Throwable {
                 if(!resource.exists())
                 {
                 	FileUtils.forceMkdir(resource.getFile());
                 }
             }
        } 
        ```

    - ThrowsAdvice

    - AfterReturningAdvice

    - Around Advice：可以实现之前提到的所有类型功能

        - 采用如下实现
        - proceed()必须调用，代表JoinPoint原本的运行逻辑

        ```java
        public interface MethodInterceptor extends Interceptor {
         	Object invoke(MethodInvocation invocation) throws Throwable;
        } 
        //
        public class PerformanceMethodInterceptor implements MethodInterceptor {
         	private final Log logger = LogFactory.getLog(this.getClass());
            
         	public Object invoke(MethodInvocation invocation) throws Throwable {
                StopWatch watch = new StopWatch();
                try{
                    watch.start();
                    return invocation.proceed();
                }
                finally
                {
                    watch.stop();
                    if(logger.isInfoEnabled())
                    {
                        logger.info(watch.toString());
                    }
                }
         	}
        }
        ```

- Introduction针对每个对象保存属性与相关逻辑

```java
public interface IntroductionInterceptor extends MethodInterceptor, DynamicIntroductionAdvice {
    
} 
```

### Aspect(Advisor)

#### PointCutAdvisor

![image-20201023110219114](D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第三部分_Spring AOP 框架.assets\image-20201023110219114.png)

- DefaultPointcutAdvisor：常规设定PointCut与Advisor
- NameMatchMethodPointcutAdvisor：限制了为PointCut为NameMatchMethodPointCut
- RegexpMethodPointcutAdvisor：正则表达式设置PoinetCut

#### IntroductionAdvisor

只能作类级别的匹配，只能使用Introduction型Advice

#### Ordered

当多个Advisor的PointCut匹配到同一处JoinPoint时，执行的先后顺序问题

### 织入

#### 各类织入器

- AspectJ：ajc编译器
- JBoss：自定义的ClassLoader
- Spring AOP：类org.springframework.aop.framework.ProxyFactory（最基本的）

![image-20201023140348517](D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第三部分_Spring AOP 框架.assets\image-20201023140348517.png)



![image-20201023140822420](D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第三部分_Spring AOP 框架.assets\image-20201023140822420.png)

#### 自动代理

AutoProxyCreator



## 第十章 Sping AOP二世

### @AspectJ形式的AOP

#### 特点

使用POJO声明Aspect和相关的Advice

新的PointCut表述方式

#### 使用

- 编写POJO
    - 编程方式注入
    - xml自动代理注入：



## 第十一章 AOP应用案例

### 异常



### 缓存