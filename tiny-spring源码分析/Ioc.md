# Ioc

## 资源获取

**tiny-spring/src/main/java/us/codecraft/tinyioc/beans/io**

```java
//Resource.java 
public interface Resource {
    InputStream getInputStream() throws IOException;
}
```

获取资源信息的输入流

```java
//ResourceLoader.java
public class ResourceLoader {
    public Resource getResource(String location){
        URL resource = this.getClass().getClassLoader().getResource(location);
        return new UrlResource(resource);
    }
}
```

3通过资源路径获取一个Resource对象

```java
//UrlResource.java
```

通过URL返回资源输入流

## Bean定义

**tiny-spring/src/main/java/us/codecraft/tinyioc/beans/**

### BeanDefinition

```java
public class BeanDefinition {

	private Object bean;

	private Class beanClass;

	private String beanClassName;

	private PropertyValues propertyValues = new PropertyValues();
    /*
    	...
    */
}
```

包装了Bean在Ioc容器中的实例，类型，属性集合

### BeanDefinitionReader

```java
public interface BeanDefinitionReader {

    void loadBeanDefinitions(String location) throws Exception;
}
```

### AbstractBeanDefinitionReader

```java
public abstract class AbstractBeanDefinitionReader implements BeanDefinitionReader {

    private Map<String, BeanDefinition> registry;

    private ResourceLoader resourceLoader;

    protected AbstractBeanDefinitionReader(ResourceLoader resourceLoader) {
        this.registry = new HashMap<String, BeanDefinition>();
        this.resourceLoader = resourceLoader;
    }

    public Map<String, BeanDefinition> getRegistry() {
        return registry;
    }

    public ResourceLoader getResourceLoader() {
        return resourceLoader;
    }
}
```

- 一个存储名-bean定义的map
- 一个读取bean配置文件的loader

### xml解析实现类

**/xml**

- 通过xml读取bean配置
- 读取配个顶级bean的配置，存入map
- 读取每个bean的ref，加入对应的propertyValues中

## 容器定义

tiny-spring/src/main/java/us/codecraft/tinyioc/beans/factory

### BeanFactory

#### BeanFactory

```java
public interface BeanFactory {

    Object getBean(String name) throws Exception;
}
```

通过名称获得Bean实例

#### AbstractBeanFactory

```java
public abstract class AbstractBeanFactory implements BeanFactory {

	private Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<String, BeanDefinition>();

	private final List<String> beanDefinitionNames = new ArrayList<String>();

	private List<BeanPostProcessor> beanPostProcessors = new ArrayList<BeanPostProcessor>();

	@Override
	public Object getBean(String name) throws Exception {
		BeanDefinition beanDefinition = beanDefinitionMap.get(name);
		if (beanDefinition == null) {
			throw new IllegalArgumentException("No bean named " + name + " is defined");
		}
		Object bean = beanDefinition.getBean();
		if (bean == null) {
			bean = doCreateBean(beanDefinition);
            bean = initializeBean(bean, name);
            beanDefinition.setBean(bean);
		}
		return bean;
	}

	protected Object initializeBean(Object bean, String name) throws Exception {
		for (BeanPostProcessor beanPostProcessor : beanPostProcessors) {
			bean = beanPostProcessor.postProcessBeforeInitialization(bean, name);
		}

		// TODO:call initialize method
		for (BeanPostProcessor beanPostProcessor : beanPostProcessors) {
            bean = beanPostProcessor.postProcessAfterInitialization(bean, name);
		}
        return bean;
	}
    /*
    	...
    */
}
```

- 一个保存bean名称-实例的map
- 获取Bean时，若不存在，则创建并保存实例至beanDefinition中
- 创建Bean同时创建property属性

#### AutowireCapableBeanFactory

```java
public class AutowireCapableBeanFactory extends AbstractBeanFactory {

	protected void applyPropertyValues(Object bean, BeanDefinition mbd) throws Exception {
		if (bean instanceof BeanFactoryAware) {
			((BeanFactoryAware) bean).setBeanFactory(this);
		}
		for (PropertyValue propertyValue : mbd.getPropertyValues().getPropertyValues()) {
			/*
				...
			*/
		}
	}
}
```

即继承抽象工厂并实现了自动装配功能

### ApplicationContext

#### ApplicationContext

```java
public interface ApplicationContext extends BeanFactory {

}
```

该接口为BeanFactory的子接口

BeanFactory需要接受Reader生成的定义Map

而该接口把 `BeanFactory` 和 `BeanDefinitionReader` 结合在了一起。

#### AbstractApplicationContext

增加refresh方法，进行BeanFactory的刷新

实现**从不同来源的不同类型的资源加载类定义**的效果。

#### ClassPathXmlApplicationContext

```java
public class ClassPathXmlApplicationContext extends AbstractApplicationContext {

	private String configLocation;

	public ClassPathXmlApplicationContext(String configLocation) throws Exception {
		this(configLocation, new AutowireCapableBeanFactory());
	}

	public ClassPathXmlApplicationContext(String configLocation, AbstractBeanFactory beanFactory) throws Exception {
		super(beanFactory);
		this.configLocation = configLocation;
		refresh();
	}

	@Override
	protected void loadBeanDefinitions(AbstractBeanFactory beanFactory) throws Exception {
		XmlBeanDefinitionReader xmlBeanDefinitionReader = new XmlBeanDefinitionReader(new ResourceLoader());
		xmlBeanDefinitionReader.loadBeanDefinitions(configLocation);
		for (Map.Entry<String, BeanDefinition> beanDefinitionEntry : xmlBeanDefinitionReader.getRegistry().entrySet()) {
			beanFactory.registerBeanDefinition(beanDefinitionEntry.getKey(), beanDefinitionEntry.getValue());
		}
	}
}
```

- 通过XmlBeanDefinitionReader解析指定路径的Resource
- 获取Bean配置信息并保存至BeanFactory

### 设计模式

*注：此处的设计模式分析不限于 tiny-spring，也包括 Spring 本身的内容*

#### 模板方法模式

该模式大量使用，例如在 `BeanFactory` 中，把 `getBean()` 交给子类实现，不同的子类 `**BeanFactory` 对其可以采取不同的实现。

#### 代理模式

在 tiny-spring 中（Spring 中也有类似但不完全相同的实现方式），`ApplicationContext` 继承了 `BeanFactory` 接口，具备了 `getBean()` 功能，但是又内置了一个 `BeanFactory` 实例，`getBean()` 直接调用 `BeanFactory` 的 `getBean()` 。但是`ApplicationContext` 加强了 `BeanFactory`，它把类定义的加载也包含进去了。











