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





## 第二十六章 基于注解开发

### 基于注解的HandlerMapping

会首先扫描应用程序的Classpath，通过反射获取所有标注了@Controller的对象，然后通过反射获取@RequestMapping的相应信息与请求信息进行匹配，并最终返回匹配后的结果。