# SpringBoot

## @SpringBootApplication

三个子注解

- @Configuration
- @EnableAutoConfiguration
- @ComponentScan

### @Configuration

 JavaConfig形式的 Spring IoC 容器的配置类使用的那个 @Configuration，使得启动类作为一个配置类

### @EnableAutoConfiguration

借助 EnableAutoConfigurationImportSelector，@EnableAutoConfiguration 可以帮助 SpringBoot 应用将所有符合条件的 @Configuration 配置都加载到当前 SpringBoot 创建并使用的 IoC 容器

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=
\org.springframework.boot.autoconfigure.admin.SpringApplicationAdmin- JmxAutoConfiguration,
\org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,
\org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,
\org.springframework.boot.autoconfigure.MessageSourceAutoConfiguration,
\org.springframework.boot.autoconfigure.PropertyPlaceholderAuto- Configuration,
\org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,
\org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,
\org.springframework.boot.autoconfigure.cassandra.CassandraAuto-Configuration,
\org.springframework.boot.autoconfigure.cloud.CloudAutoConfiguration,
\org.springframework.boot.autoconfigure.context.ConfigurationProperties-AutoConfiguration,
\org.springframework.boot.autoconfigure.dao.PersistenceException-TranslationAutoConfiguration,
\org.springframework.boot.autoconfigure.data.cassandra.Cassandra-DataAutoConfiguration,
\org.springframework.boot.autoconfigure.data.cassandra.Cassandra-RepositoriesAutoConfiguration,
\...
```

从 classpath 中搜寻所有 META-INF/spring.factories 配置文件，并将其中 org.spring-framework.boot.autoconfigure.EnableAutoConfiguration 对应的配置项通过反射（Java Reflection）实例化为对应的标注了 @Configuration 的 JavaConfig 形式的 IoC 容器配置类，然后汇总为一个并加载到 IoC 容器。

## 事务传播行为

requires,supports,mandatory,requires_new,not_supported,never,nested

## 懒加载

@Lazy，使用该注解使得IOC容器(ApplicationContext)在该Bean需要使用的时候才将其实例化。

## 静态属性注入

Spring不支持静态变量的依赖注入，会发生空指针异常。

原因：Spring是基于对象层面的依赖注入，而静态变量不是对象的属性而是一个类的属性，是一种global全局状态。

### 注入方法

- setter方法注入
	- 在setter方法上添加注解
	- 在xml文件中配置
	- 再setter方法中使用类.xxx = xxx
- @postConstruct注解（注册init-method方法)

```java
import org.mongodb.morphia.AdvancedDatastore;  
import org.springframework.beans.factory.annotation.Autowired;  
  
  
@Component  
public class MongoFileOperationUtil {  
    @Autowired  
    private static AdvancedDatastore dsForRW;  
  
    private static MongoFileOperationUtil mongoFileOperationUtil;  
  
    @PostConstruct  
    public void init() {  
        mongoFileOperationUtil = this;  
        mongoFileOperationUtil.dsForRW = this.dsForRW;  
    }  
  
}
```

## 循环依赖

### 意义



### 三级缓存

1. `singletonObjects`，一级缓存，存储的是所有创建好了的单例Bean
2. `earlySingletonObjects`，完成实例化，但是还未进行属性注入及初始化的对象
3. `singletonFactories`，提前暴露的一个单例工厂，二级缓存中存储的就是从这个工厂中获取到的对象

当A、B两个类发生循环引用时，在A完成实例化后，就使用实例化后的对象去创建一个对象工厂，并添加到三级缓存中。

当A进行属性注入时，会去创建B，同时B又依赖了A，所以创建B的同时又会去调用getBean(a)来获取需要的依赖，此时的getBean(a)会从缓存中获取

