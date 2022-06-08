# 目录

[TOC]

# 概述

# IOC和DI
## 容器对象
`Spring` 容器对象有 `ApplicationContext` 和 `BeanFactory`，`ApplicationContext` 包含 `BeanFactory` 的所有功能，建议优先使用 `ApplicationContext`。

### FactoryBean
一般情况下，Spring通过反射机制利用<bean>的class属性指定实现类实例化Bean，在某些情况下，实例化Bean过程比较复杂，如果按照传统的方式，则需要在<bean>中提供大量的配置信息。配置方式的灵活性是受限的，这时采用编码的方式可能会得到一个简单的方案。Spring为此提供了一个org.springframework.bean.factory.FactoryBean的工厂类接口，用户可以通过实现该接口定制实例化Bean的逻辑。FactoryBean接口对于Spring框架来说占用重要的地位，Spring自身就提供了70多个FactoryBean的实现。

```java
@Service
public class DefaultFactoryBean implements FactoryBean<SchoolBean> {
    private String name = "default factory bean";

    @Override
    public SchoolBean getObject() {
        return new SchoolBean(1L, "第一中学");
    }

    @Override
    public Class<?> getObjectType() {
        return SchoolBean.class;
    }
}
```

#### 源码解析
```java
public interface FactoryBean<T> {
    String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";
    
    /**
     * 返回由FactoryBean创建的Bean实例，
     * 如果isSingleton()返回true，则该实例会放到Spring容器中单实例缓存池中；
     */
    @Nullable
    T getObject() throws Exception;
    
    /**
     * 返回FactoryBean创建的Bean类型。
     */
    @Nullable
    Class<?> getObjectType();
    
    /**
     * 返回由FactoryBean创建的Bean实例的作用域是singleton还是prototype
     */
    default boolean isSingleton() {
        return true;
    }
}
```

### BeanFactory
BeanFactory是IOC容器的核心接口，它的职责包括：实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。BeanFactory只是个接口，并不是IOC容器的具体实现，但是Spring容器给出了很多种实现，如 DefaultListableBeanFactory、XmlBeanFactory、ApplicationContext等，其中XmlBeanFactory就是常用的一个，该实现将以XML方式描述组成应用的对象及对象间的依赖关系。

原始的BeanFactory无法支持spring的许多插件，如AOP功能、Web应用等。

#### 实现 `BeanFactoryAware` 接口获取BeanFactory
```java
@Service
public class ObtainContextByBeanFactoryAware implements BeanFactoryAware {
    private BeanFactory beanFactory;

    @Override
    public void setBeanFactory(@NonNull BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
    }

    public String getContent() {
        ContextBean contextBean = (ContextBean)beanFactory.getBean("contextBean");
        return contextBean.getContent();
    }
}
```

#### `BeanFactory` 和 `FactoryBean` 的区别
`BeanFactory` 是接口，提供了IOC容器最基本的形式，给具体的IOC容器的实现提供了规范；

`FactoryBean` 也是接口，为IOC容器中Bean的实现提供了更加灵活的方式，`FactoryBean` 在IOC容器的基础上给Bean的实现加上了一个简单工厂模式和装饰模式，可以在getObject()方法中灵活配置。

```java
@Service
public class DefaultBeanFactory implements BeanFactoryAware {
    private BeanFactory beanFactory;

    @Override
    public void setBeanFactory(@NonNull BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
    }

    public void execute() {
        String beanName = "defaultFactoryBean";
        Object bean = beanFactory.getBean(beanName);
        if(bean instanceof SchoolBean) {
            System.err.println("bean类型是SchoolBean");
        }
        Object factoryBean = beanFactory.getBean(BeanFactory.FACTORY_BEAN_PREFIX + beanName);
        if(factoryBean instanceof FactoryBean) {
            System.err.println("bean类型是FactoryBean");
        }
    }
}
```

#### 源码解析
```java
public interface BeanFactory {  
    String FACTORY_BEAN_PREFIX = "&";

    /**
     * 返回给定名称注册的bean实例。根据bean的配置情况，如果是singleton模式将返回一个共享实例，
     * 否则将返回一个新建的实例，如果没有找到指定bean,该方法可能会抛出异常
     */
    Object getBean(String name) throws BeansException;

    /**
     * 返回以给定名称注册的bean实例，并转换为给定class类型
     */
    <T> T getBean(String name, Class<T> requiredType) throws BeansException;
    Object getBean(String name, Object... args) throws BeansException;
    <T> T getBean(Class<T> requiredType) throws BeansException;
    <T> T getBean(Class<T> requiredType, Object... args) throws BeansException;
    <T> ObjectProvider<T> getBeanProvider(Class<T> requiredType);
    <T> ObjectProvider<T> getBeanProvider(ResolvableType requiredType);

    /**
     * 判断工厂中是否包含给定名称的bean定义，若有则返回true
     */
    boolean containsBean(String name);

    /**
     * 判断给定名称的bean定义是否为单例模式
     */
    boolean isSingleton(String name) throws NoSuchBeanDefinitionException;  boolean isPrototype(String name) throws NoSuchBeanDefinitionException;  boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;  boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;    

    /**
     * 返回给定名称的bean的Class,
     * 如果没有找到指定的bean实例，则抛出NoSuchBeanDefinitionException异常
     */
    @Nullable
    Class<?> getType(String name) throws NoSuchBeanDefinitionException;

    /**
     * 返回给定bean名称的所有别名 
     */
    @Nullable
    Class<?> getType(String name, boolean allowFactoryBeanInit) throws NoSuchBeanDefinitionException;

    /**
     * 返回给定bean名称的所有别名 
     */
    String[] getAliases(String name);
}
```

### ApplicationContext 
 
ApplicationContext以一种更向面向框架的方式工作以及对上下文进行分层和实现继承，ApplicationContext包还提供了以下的功能： 

- MessageSource, 提供国际化的消息访问 
- 资源访问，如URL和文件 
- 事件传播 
- 载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层; 

#### 实现 `ApplicationContextAware` 接口
```java
@Service
public class ObtainContextByApplicationContextAware implements ApplicationContextAware {
    private ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(@Nonnull ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    public String getContent() {
        ContextBean contextBean = (ContextBean)applicationContext.getBean("contextBean");
        return contextBean.getContent();
    }
}
```



#### 实现 `ApplicationListener` 接口
```java
@Service
public class ObtainContextByApplicationListener implements ApplicationListener<ContextRefreshedEvent> {
    private ApplicationContext applicationContext;

    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        this.applicationContext = event.getApplicationContext();
    }

    public String getContent() {
        ContextBean contextBean = (ContextBean)applicationContext.getBean("contextBean");
        return contextBean.getContent();
    }
}
```


## 对象属性注入
### `@Autowired` 注解
- 属性注入

属性注入非常简洁，没有任何多余代码，非常有效的提高了java的简洁性。即使再多几个依赖一样能解决掉这个问题。

- setter方法注入

在使用set方式时，这是一种选择注入，可有可无，即使没有注入这个依赖，那么也不会影响整个类的运行。

- 构造器注入

在使用构造器方式时已经显式注明必须强制注入。通过强制指明依赖注入来保证这个类的运行。

变量方式注入应该尽量避免，使用set方式注入或者构造器注入，这两种方式的选择就要看这个类是强制依赖的话就用构造器方式，选择依赖的话就用set方法注入。
## 对象初始化
### xml中的 `init-method` 配置
这种方式现在已经很少使用，推荐下面两种初始化方式。



### 实现 `InitializingBean` 接口
```java
@Slf4j
@Service
public class InitByInterface implements InitializingBean {
    @Override
    public void afterPropertiesSet() {
        log.info("通过实现InitializingBean接口来进行初始化");
    }
}
```



### `@PostConstruct` 注解
```java
@Slf4j
@Service
public class InitByAnnotation {
    @PostConstruct
    public void init() {
        log.info("通过@PostConstruct注解进行初始化操作");
    }
}
```

#### 源码解析
> org.springframework.beans.factory.config.BeanPostProcessor
```java
public interface BeanPostProcessor {
	@Nullable
	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

	@Nullable
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
}
```

> org.springframework.context.annotation.CommonAnnotationBeanPostProcessor
```java
public class CommonAnnotationBeanPostProcessor extends InitDestroyAnnotationBeanPostProcessor
		implements InstantiationAwareBeanPostProcessor, BeanFactoryAware, Serializable {
    public CommonAnnotationBeanPostProcessor() {
        setOrder(Ordered.LOWEST_PRECEDENCE - 3);
        // 调用父类方法设置被@PostConstruct注解
        setInitAnnotationType(PostConstruct.class);
        setDestroyAnnotationType(PreDestroy.class);
        ignoreResourceType("javax.xml.ws.WebServiceContext");
    }
}
```

> org.springframework.beans.factory.annotation.InitDestroyAnnotationBeanPostProcessor
```java
public class InitDestroyAnnotationBeanPostProcessor
		implements DestructionAwareBeanPostProcessor, MergedBeanDefinitionPostProcessor, PriorityOrdered, Serializable {
    // 通过CommonAnnotationBeanPostProcessor设置initAnnotationType为@PostConstruct
    public void setInitAnnotationType(Class<? extends Annotation> initAnnotationType) {
        this.initAnnotationType = initAnnotationType;
    }
    
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        // 获取初始化和销毁方法
        LifecycleMetadata metadata = findLifecycleMetadata(bean.getClass());
        try {
            // 通过反射执行初始化方法
            metadata.invokeInitMethods(bean, beanName); 
        }
        // ...
        // 执行完初始化方法后返回对象并执行下一个BeanPostProcessor的初始化方法
        return bean;
    } 

    private LifecycleMetadata buildLifecycleMetadata(final Class<?> clazz) {
        // ...
        List<LifecycleElement> initMethods = new ArrayList<>();
        // ...
        Class<?> targetClass = clazz;
        do {
            final List<LifecycleElement> currInitMethods = new ArrayList<>();
            // ...
            ReflectionUtils.doWithLocalMethods(targetClass, method -> {
                if (this.initAnnotationType != null && method.isAnnotationPresent(this.initAnnotationType)) {
                    LifecycleElement element = new LifecycleElement(method);
                    // 获取被@PostConstruct标注的方法并将其添加到
                    currInitMethods.add(element);
                    // ...
                }
                // ...
            });
            initMethods.addAll(0, currInitMethods);
            // ...
            targetClass = targetClass.getSuperclass();
        }
        while (targetClass != null && targetClass != Object.class);
        return (initMethods.isEmpty() && destroyMethods.isEmpty() ? this.emptyLifecycleMetadata :
                new LifecycleMetadata(clazz, initMethods, destroyMethods));
    }
}
```

### 对象初始化顺序
初始化方式的顺序如下：

Constructor构造方法 -> `@Autowired` -> `@PostConstruct` -> `InitializingBean` -> `init-method`
```java
@Slf4j
@Service
public class InitOrder implements InitializingBean {
    private ContextBean contextBean;
    public InitOrder() {
        log.info("1、先执行构造器初始化");
        if(contextBean == null) {
            log.info("此时contextBean为null");
        }
    }

    @Autowired
    public void setContextBean(ContextBean contextBean) {
        log.info("2、再执行@Autowired注解初始化");
        this.contextBean = contextBean;
        if(contextBean != null) {
            log.info("此时contextBean不为null");
        }
    }

    @PostConstruct
    public void init() {
        log.info("3、再执行@PostConstruct注解初始化");
    }

    @Override
    public void afterPropertiesSet() {
        log.info("4、最后执行afterPropertiesSet方法和init-method进行初始化");
    }
}
```

#### 源码解析
决定他们调用顺序的关键代码在 `AbstractAutowireCapableBeanFactory` 类的 `initializeBean` 方法中。

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory
```java
 abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory
		implements AutowireCapableBeanFactory {
    protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
        // ...
        Object wrappedBean = bean;
        if (mbd == null || !mbd.isSynthetic()) {
            // 调用BeanPostProcessor，注解@PostConstruct就是通过InitDestroyAnnotationBeanPostProcessor实现的
            wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
        }
    
        try {
            // 调用初始化方法，包括afterPropertiesSet和init-method
            invokeInitMethods(beanName, wrappedBean, mbd);
        }
        // ...
    }

    @Override
    public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
            throws BeansException {
        Object result = existingBean;
        for (BeanPostProcessor processor : getBeanPostProcessors()) {
            // 调用BeanPostProcessor的postProcessBeforeInitialization进行初始化
            Object current = processor.postProcessBeforeInitialization(result, beanName);
            if (current == null) {
                return result;
            }
            result = current;
        }
        return result;
    }

    protected void invokeInitMethods(String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
        // 先调用InitializingBean接口的afterPropertiesSet方法初始化对象
        boolean isInitializingBean = (bean instanceof InitializingBean);
        if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
            // ...
            if (System.getSecurityManager() != null) {
                try {
                    AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
                        ((InitializingBean) bean).afterPropertiesSet();
                        return null;
                    }, getAccessControlContext());
                }
                // ...
            }
            else {
                ((InitializingBean) bean).afterPropertiesSet();
            }
        }
    
        // 再调用init-method方法初始化对象
        if (mbd != null && !Objects.equals(bean.getClass(), NullBean.class)) {
            String initMethodName = mbd.getInitMethodName();
            if (StringUtils.hasLength(initMethodName) &&
                    !(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
                    !mbd.isExternallyManagedInitMethod(initMethodName)) {
                invokeCustomInitMethod(beanName, bean, mbd);
            }
        }
    }
}
```

## 对象作用域
，有几个范围可选的：
- singleton：单例；
- prototype：多例；
- request：在一次http请求内有效；
- session：在一个用户会话内有效；
- globalSession：在全局会话内有效。

### 五种作用域
singleton和prototype作用域适用于所有场景的Spring容器，request、session和globalSession作用域仅适用于Web场景下的Spring容器。

#### singleton
Springboot的注入默认范围是单例，在容器中通过对象引入配置注入和通过容器的getBean()方法返回的实例都是同一个bean。容器在启动的时候，自动实例化所有的singleton的bean并缓存于容器当中。

- 优点

1、对bean提前的实例化操作，会及早发现一些潜在的配置的问题。

2、Bean以缓存的方式运行，当运行到需要使用该bean的时候，就不需要再去实例化了。加快了运行效率。

#### prototype
是指每次从容器中调用Bean时，都返回一个新的实例，即每次调用getBean()时，相当于执行new Bean()的操作。在默认情况下，容器在启动时不实例化prototype的Bean，容器将prototype的实例交给调用者之后便不在管理他的生命周期了。

对于有状态的Bean应该使用prototype，对于无状态的Bean则使用singleton

#### request
对应一个http请求和生命周期，当http请求调用作用域为request的bean的时候,Spring便会创建一个新的bean，在请求处理完成之后便及时销毁这个bean。

#### session
Session中所有http请求共享同一个请求的bean实例。Session结束后就销毁bean。

#### globalSession
与session大体相同，但仅在portlet应用中使用。Portlet规范定义了全局session的概念。请求的bean被组成所有portlet的自portlet所共享。如果不是在portlet这种应用下，globalSession则等价于session作用域。

### @Scope
### @Scope注解衍生注解
`@RequestScope`, `@SessionScope` 和 `@ApplicationScope` 注解都是 `@Scope` 注解的衍生注解，其功能相当于设置 `@Scope` 注解的对应 `value` 属性。



> org.springframework.web.context.annotation.RequestScope
```java
@Scope(WebApplicationContext.SCOPE_REQUEST)
public @interface RequestScope {
	// ...
}
```



> org.springframework.web.context.annotation.SessionScope
```java
@Scope(WebApplicationContext.SCOPE_SESSION)
public @interface SessionScope {
	// ...
}
```



> org.springframework.web.context.annotation.ApplicationScope 
```java
@Scope(WebApplicationContext.SCOPE_APPLICATION)
public @interface ApplicationScope {
	// ...
}
```
## 对象生命周期

## SpEL表达式

## 常用注解

# AOP

## 基础概念

## AOP实践

# DataAccess

# 名称空间

# 常用类

# 整合Web

# spring模块
## spring-beans
### Aware接口
`Aware` 是一个空接口，里面不包括任何方法。该接口表示已感知，可以通过该接口的实现获取指定对象。

> org.springframework.beans.factory.Aware
```java
public interface Aware {
}
```

该接口的部分实现如下：

- ApplicationEventPublisherAware 
- ServletContextAware 
- MessageSourceAware 
- ResourceLoaderAware 
- SchedulerContextAware 
- NotificationPublisherAware 
- EnvironmentAware 
- BeanFactoryAware 
- EmbeddedValueResolverAware 
- ImportAware 
- ServletConfigAware 
- BootstrapContextAware 
- LoadTimeWeaverAware
- BeanNameAware
- BeanClassLoaderAware
- ApplicationContextAware



# 参考资料
- [spring-Sessions are not supported by the MongoDB cluster to which this client is connected](https://blog.csdn.net/ssehs/article/details/105301345)