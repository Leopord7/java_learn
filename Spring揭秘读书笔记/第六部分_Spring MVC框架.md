# Spring MVC框架

## 第二十二章 迈向Spring MVC的旅程



## 第二十三章 SpringMVC初体验

### DispatcherServlet的处理流程

![image-20201026100722169](D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第六部分_Spring MVC框架.assets\image-20201026100722169.png)

### 实例

#### 传统web项目结构

![image-20201026102342758](D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第六部分_Spring MVC框架.assets\image-20201026102342758.png)

#### 配置文件

- ContextLoaderListener与/WEB-INF/applicationContext.xml
    - 提供顶层的WebApplicationContext
-  DispatcherServlet与XXX-servlet.xml 
    - 主要负责配置基于Spring MVC框架的Web应用程序所使用的各种Web层组件
    - DispatcherServlet启动之后将加载-servlet.xml配置文件，并构建相应的 WebApplicationContext。该WebApplicationContext将之前通过ContextLoaderListener加载的顶层WebApplicationContext（ROOT WebApplicationContext）作为父容器（Parent ApplicationContext）。
    - 可以配置多个controller配置文件

![image-20201026171507048](D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第六部分_Spring MVC框架.assets\image-20201026171507048.png)

![image-20201026171516034](D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第六部分_Spring MVC框架.assets\image-20201026171516034.png)

![image-20201026171523927](D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第六部分_Spring MVC框架.assets\image-20201026171523927.png)

## 第二十四章 近距离接触各个角色



## 第二十五章 认识更多成员

### Handler与HandlerAdapter

#### 含义

dispatchservlet如何知道调用什么类型的Handler，调用Handler的哪个方法？

不同Handler的调用职责转交给HandlerAdapter

```java
public interface HandlerAdapter {
 boolean supports(Object handler);
 ModelAndView handle(HttpServletRequest request, HttpServletResponse response, ➥
 Object handler) throws Exception;
 long getLastModified(HttpServletRequest request, Object handler);
} 
```

DispatcherServlet从HandlerMapping获得一个Handler之后，将询问 HandlerAdaptor的supports(..)方法，以便了解当前HandlerAdaptor是否支持HandlerMapping刚刚返回的Handler类型的调用。如果supports(..)返回true，DispatcherServlet 则 调用 HandlerAdaptor的handle(..)方法，同时将刚才的Handler作为参数传入。方法执行后将返回 ModelAndView，之后的工作就由ViewResolver接手了。

DispatchServlet不需要面对众多不同类的Handler了，Adapter起到了隔离作用

### 框架内处理流程拦截与HandlerInterceptor

#### 含义

HandlerExecutionChain就是一个数据载体，它包含了两方面的数据，一个就是用于处 理Web请求的Handler，另一个则是一组随同Handler一起返回的HandlerInterceptor。

#### 归属

HandlerInterceptor→HandlerExecutionChain→HandlerMapping的顺序溯源而上，HandlerMapping的setInterceptors方法



## 第二十六章 基于注解开发

### 基于注解的HandlerMapping

会首先扫描应用程序的Classpath，通过反射获取所有标注了@Controller的对象，然后通过反射获取@RequestMapping的相应信息与请求信息进行匹配，并最终返回匹配后的结果。