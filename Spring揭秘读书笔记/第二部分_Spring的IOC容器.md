# Spring的IoC容器

## 第二章 IoC的基本概念

### 概念

如果我们依赖某个服务，最直接的方式就是直接新建相应的依赖类；但实际上我们只需要在需要这项服务时，相应对象可以准备就绪，没有亲手创建的必要。

IoC容器做的就是在被注入对象需要的时候将对应的依赖对象进行注入。

#### 控制反转

取得被依赖对象的控制方由被注入对象变为了IoC容器。

#### 依赖注入

IoC容器在被注入对象需要的时候注入相应的依赖对象。

### 注入方式(被注入对象通知IoC容器)

#### 构造方法注入

- 优点：对象构造完成后进入就绪状态
- 缺点：当依赖对象增多与非依赖对象的处理难以维护

#### setter方法注入

- 优点：描述清晰
- 缺点：无法进入就绪状态

#### 接口注入(退役状态)

### 优点总结

IoC是一种帮助我们解耦各业务对象间依赖关系的对象绑定方式

1. 不用自己组装，拿来就用。
2. 享受单例的好处，效率高，不浪费空间。
3. 便于单元测试，方便切换mock组件。
4. 便于进行AOP操作，对于使用者是透明的。
5. 统一配置，便于修改

## 第三章 掌管大局的IoC Service Provider

### IoC的职责

构建业务对象以及进行依赖绑定

### 管理方式

配置文件

## 第四章 Spring的IoC容器之BeanFactory

### 特点

- 基础型IoC容器
- 懒加载策略，当访问某个注册对象的的时候，容器才初始化该对象并注入相应依赖，初期启动速度较快
- 所需系统资源有限，适用于功能不是很严格场景

### 对象注册与绑定方式

#### 直接编码方式



#### 从配置文件读取

properties配置文件或xml配置文件

使用相应的方法实现reader接口，得到definition存储与registerMap中，registerMap用于beanFactory的初始化

### XML注册规则

#### 关键字

id	class	name

#### 构造器注入

![image-20201021125547870](D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第一部分_Spring的IOC容器.assets\image-20201021125547870.png)

#### setter注入

![image-20201021125631883](D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第一部分_Spring的IOC容器.assets\image-20201021125631883.png)

#### < property > < constructor-arg >配置项

![image-20201021125844030](D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第一部分_Spring的IOC容器.assets\image-20201021125844030.png)



![image-20201021125849337](D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第一部分_Spring的IOC容器.assets\image-20201021125849337.png)

#### 其他

- depends-on
- autowire
- dependency-check
- lazy-init

#### 继承与模板化配置



#### 作用域

- singleton
- prototype
- request、session和global session 

### 工厂方法的Bean注入

当依赖的接口实现类为第三方库时使用

#### 静态工厂

![image-20201021142447286](D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第一部分_Spring的IOC容器.assets\image-20201021142447286.png)



#### 非静态工厂

![image-20201021142632963](D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第一部分_Spring的IOC容器.assets\image-20201021142632963.png)

![image-20201021142638157](D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第一部分_Spring的IOC容器.assets\image-20201021142638157.png)

#### FactoryBean

FactoryBean是Spring容器提供的一种可以扩展容器对象实例化逻辑的接口

这种类型的Bean本身就是生产对象的工厂 （Factory）

![image-20201021143531855](D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第一部分_Spring的IOC容器.assets\image-20201021143531855.png)



![image-20201021143536941](D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第一部分_Spring的IOC容器.assets\image-20201021143536941.png)

NextDayDateDisplayer所声明的依赖dateOfNextDay的类型为DateTime，而不是 NextDayDateFactoryBean。也就是说FactoryBean类型的bean定义，通过正常的id引用，容器返回 的是FactoryBean所“生产”的对象类型，而非FactoryBean实现本身。

### 偷梁换柱

声明为非单例的Bean可能由于保持最初的引用而不会再次获取

#### 方法注入

![image-20201021144403657](D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第一部分_Spring的IOC容器.assets\image-20201021144403657.png)

#### 引用容器本身

关键是实现BeanFactoryAware接口

![image-20201021144935504](D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第一部分_Spring的IOC容器.assets\image-20201021144935504.png)

### 容器运行

#### 容器启动

分为容器启动阶段和Bean实例化阶段

- xml/property -->  beanDefinition --> beanRegistry
- getBean --> 创建实例 --> 注入依赖

#### 插手容器启动

BeanFactoryPostProcessor在第一阶段即将结束时对beanDefinition进行一些额外操作

BeanFactory需要手动应用 ApplicationContext会自动识别BeanFactoryPostProcesser并应用

##### 实例

PropertyPlaceholderConfigurer 会将xml中的${} (property配置文件)转换为真实数据

#### Bean的一生

##### getBean

第一次调用getBean时会开始Bean的实例化(非prototype)

BeanFactory采用懒加载 ApplicationContext会在启动后实例化所有Bean实例

##### Bean的实例化

- 反射或cglib动态字节码生成为手段根据bean定义返回bean实例
- 用BeanWrapper对对象实例进行包裹以统一方式访问，并进行类型转换和设置属性值，返回对应的BeanWrapper实例
- BeanWrapper借助于propertyEditor完成转换对象类型，设置属性值工作

##### Aware接口

将aware接口中定义的依赖注入实例，如BeanFactoryAware注入BeanFactory实例

##### BeanPostProcessor

有after和before的处理

- 处理标记接口实现类，如aware接口
- 字节码增强
- 生成代理对象

<img src="D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第二部分_Spring的IOC容器.assets\image-20201027122006620.png" alt="image-20201027122006620" style="zoom:67%;" />

<img src="D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第二部分_Spring的IOC容器.assets\image-20201027122019340.png" alt="image-20201027122019340" style="zoom:67%;" />

##### InitializingBean和init-method

在BeanPostProcessor的前置处理后

如果实现了InitializingBean的void afterPropertiesSet() throws Exception; 方法

或使用了<bean>的<init-method>属性

会在后置处理前调用该方法修改Bean状态

##### DisposableBean与destroy-method

销毁回调，与前一项对应

## 第五章 Spring IoC容器 ApplicationContext

### 统一资源加载策略

#### Resource

Spring框架内部使用org.springframework.core.io.Resource接口作为所有资源的抽象和访问接口

ByteArray、ClassPath、FileSystem、URL、InputStream

#### ResourceLoader

查找和定位Resource，核心方法getResource()

![image-20201021163440355](D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第一部分_Spring的IOC容器.assets\image-20201021163440355.png)

![image-20201022093603863](D:\huangchenhong\note\java_learn\Spring揭秘读书笔记\第一部分_Spring的IOC容器.assets\image-20201022093603863.png)

- ApplicationContext继承自ResourcePatternResolver
- AbstractApplicationContext继承自DefaultResourceLoader
- AbstractApplicationContext有一个PathMatchingResourcePatternResolver实例
- PathMatchingResourcePatternResolver实例在传ResourceLoader的时候即指定传入自己

以此同时实现了ResourceLoader和ResourcePatternResolver，即实现了**资源统一加载**

**对比BeanFactory**：通过不同的BeanDefinitionReader实现类来加载资源

#### 对外表现

- 可作为ResourceLoader使用
- 需要注入ResourceLoader的Bean可通过实现ResourceLoaderAware或者ApplicationContextAware接口，直接将ApplicationContext自身注入
- 可以实现Resource类型的依赖注入：注册ResourceEditor
- ClassPathXmlApplicationContext 默认从classPath查找资源  FileSystemXmlApplicationContext 默认从文件系统查找

### 国际化信息支持



### 事件发布

#### javaSE的事件发布



#### Spring的事件发布

##### 相关类

- ApplicationEvent：Spring容器自定义事件类型，有ContextClosedEvent，ContextRefreshedEvent等
- ApplicationListener：ApplicationContext容器在启动时，会自动识别并加载EventListener类型bean定义， 一旦容器内有事件发布，将通知这些注册到容器的EventListener。
- ApplicationContext
  - 继承了ApplicationEventPublisher接口，承担事件发布者的角色
  - 并不亲自做，交给内部实现了ApplicationEventMulticaster接口的实现类来做

##### 具体使用

让对象支持事件发布，首先向目标对象注入实现了ApplicationEventPublisher的实例，即ApplicationContext本身

- 对象使用ApplicationEventPublisherAware接口
- 对象实现ApplicationContextAware接口

最后将该对象和Listener注入到ApplicationContext中

### 多配置模块加载的简化

可通过通配符路径或是String路径数组同时加载多个资源

### 基于注解的自动绑定

注解的实质是注册了BeanPostProcessor类型的Bean

### 将革命进行得更彻底一些（classpath-scanning 功能介绍）

