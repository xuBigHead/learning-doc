# Spring总览

Spring 是一种轻量级开发框架，旨在提高开发人员的开发效率以及系统的可维护性。



## Spring组成

Spring框架至今已集成了20多个模块，这些模块分布在以下模块中：

- 核心容器（Core Container）
- 数据访问/集成（Data Access/Integration）层
- Web层
- AOP（Aspect Oriented Programming）模块
- 植入（Instrumentation）模块
- 消息传输（Messaging）
- 测试（Test）模块

![1](../../Image/2021/04/210407.png)



| 组成部分                      | 概述                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| Core                          | 提供了框架的基本组成部分，包括控制反转（Inversion of Control，IOC）和依赖注入（Dependency Injection，DI）功能。 |
| Beans                         | 提供了BeanFactory，是工厂模式的一个经典实现，Spring将管理对象称为Bean。 |
| Context                       | 建立在Core和Beans模块的基础之上，提供一个框架式的对象访问方式，是访问定义和配置的任何对象的媒介。ApplicationContext接口是Context模块的焦点。 |
| Context-Support               | 持整合第三方库到Spring应用程序上下文，特别是用于高速缓存（EhCache、JCache）和任务调度（CommonJ、Quartz）的支持。 |
| Expression                    | 提供了强大的表达式语言去支持运行时查询和操作对象图。这是对JSP2.1规范中规定的统一表达式语言（Unified EL）的扩展。该语言支持设置和获取属性值、属性分配、方法调用、访问数组、集合和索引器的内容、逻辑和算术运算、变量命名以及从Spring的IOC容器中以名称检索对象。它还支持列表投影、选择以及常用的列表聚合。 |
| AOP                           | 提供了一个符合AOP要求的面向切面的编程实现，允许定义方法拦截器和切入点，将代码按照功能进行分离，以便干净地解耦。 |
| Aspects                       | 提供了与AspectJ的集成功能，AspectJ是一个功能强大且成熟的AOP框架。 |
| Instrument                    | 提供了类植入（Instrumentation）支持和类加载器的实现，可以在特定的应用服务器中使用。 |
| Messaging                     | 该模块提供了对消息传递体系结构和协议的支持。                 |
| JDBC                          | 提供了一个JDBC的抽象层，消除了烦琐的JDBC编码和数据库厂商特有的错误代码解析。 |
| orm                           | 为流行的对象关系映射（Object-Relational Mapping）API提供集成层，包括JPA和Hibernate。使用Spring-orm模块可以将这些O/R映射框架与Spring提供的所有其他功能结合使用，例如声明式事务管理功能。 |
| oxm                           | 提供了一个支持对象/XML映射的抽象层实现，例如JAXB、Castor、JiBX和XStream。 |
| jms（Java Messaging Service） | 指Java消息传递服务，包含用于生产和使用消息的功能。自Spring4.1以后，提供了与Spring-messaging模块的集成。 |
| tx                            | 支持用于实现特殊接口和所有POJO（普通Java对象）类的编程和声明式事务管理。 |
| web                           | 提供了基本的Web开发集成功能，例如多文件上传功能、使用Servlet监听器初始化一个IOC容器以及Web应用上下文。 |
| webmvc                        | 也称为Web-Servlet模块，包含用于web应用程序的Spring MVC和REST Web Services实现。Spring MVC框架提供了领域模型代码和Web表单之间的清晰分离，并与Spring Framework的所有其他功能集成。 |
| websocket                     | Spring4.0以后新增的模块，它提供了WebSocket和SocketJS的实现。 |
| Portlet                       | （已废弃）类似于Servlet模块的功能，提供了Portlet环境下的MVC实现。 |
| Spring-test                   | 支持使用JUnit或TestNG对Spring组件进行单元测试和集成测试。    |



## 总结

### 缺点

Spring的配置是重量级的，需要大量的XML配置。



### 设计模式应用

- **工厂设计模式** : Spring使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。
- **代理设计模式** : Spring AOP 功能的实现。
- **单例设计模式** : Spring 中的 Bean 默认都是单例的。
- **包装器设计模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
- **观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。
- **适配器模式** :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配`Controller`。



# Spring Core

## IOC和DI

IoC（Inverse of Control:控制反转）是一种**设计思想**，就是 **将原本在程序中手动创建对象的控制权，交由Spring框架来管理。** IoC 在其他语言中也有应用，并非 Spring 特有。 **IoC 容器是 Spring 用来实现 IoC 的载体， IoC 容器实际上就是个Map（key，value）,Map 中存放的是各种对象。**

 **IoC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。**



### 容器对象
`Spring` 容器对象有 `ApplicationContext` 和 `BeanFactory`，`ApplicationContext` 包含 `BeanFactory` 的所有功能，建议优先使用 `ApplicationContext`。



#### FactoryBean
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

##### 源码解析
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

#### BeanFactory
BeanFactory是IOC容器的核心接口，它的职责包括：实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。BeanFactory只是个接口，并不是IOC容器的具体实现，但是Spring容器给出了很多种实现，如 DefaultListableBeanFactory、XmlBeanFactory、ApplicationContext等，其中XmlBeanFactory就是常用的一个，该实现将以XML方式描述组成应用的对象及对象间的依赖关系。

原始的BeanFactory无法支持spring的许多插件，如AOP功能、Web应用等。

##### 实现 `BeanFactoryAware` 接口获取BeanFactory
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

##### `BeanFactory` 和 `FactoryBean` 的区别
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

##### 源码解析
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

#### ApplicationContext 

ApplicationContext以一种更向面向框架的方式工作以及对上下文进行分层和实现继承，ApplicationContext包还提供了以下的功能： 

- MessageSource, 提供国际化的消息访问 
- 资源访问，如URL和文件 
- 事件传播 
- 载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层; 

##### 实现 `ApplicationContextAware` 接口
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



##### 实现 `ApplicationListener` 接口
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


### 对象属性注入
#### `@Autowired` 注解
- 属性注入

属性注入非常简洁，没有任何多余代码，非常有效的提高了java的简洁性。即使再多几个依赖一样能解决掉这个问题。

- setter方法注入

在使用set方式时，这是一种选择注入，可有可无，即使没有注入这个依赖，那么也不会影响整个类的运行。

- 构造器注入

在使用构造器方式时已经显式注明必须强制注入。通过强制指明依赖注入来保证这个类的运行。

变量方式注入应该尽量避免，使用set方式注入或者构造器注入，这两种方式的选择就要看这个类是强制依赖的话就用构造器方式，选择依赖的话就用set方法注入。
### 对象初始化
#### xml中的 `init-method` 配置
这种方式现在已经很少使用，推荐下面两种初始化方式。



#### 实现 `InitializingBean` 接口
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



#### `@PostConstruct` 注解
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

##### 源码解析
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

#### 对象初始化顺序
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

##### 源码解析
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



#### @Scope
#### @Scope注解衍生注解
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



# Spring Bean

## 对象

### 声明对象方式

一般使用 `@Autowired` 注解自动装配 bean，要想把类标识成可用于 `@Autowired` 注解自动装配的 bean 的类，采用以下注解可实现：

- `@Component` ：通用的注解，可标注任意类为 `Spring` 组件。如果一个Bean不知道属于哪个层，可以使用`@Component` 注解标注。
- `@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。
- `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao层。
- `@Controller` : 对应 Spring MVC 控制层，主要用户接受用户请求并调用 Service 层返回数据给前端页面。



### 对象作用域

| 作用域        | 描述                 | 适用场景 |
| ------------- | -------------------- | -------- |
| singleton     | 单例                 | 所有场景 |
| prototype     | 多例                 | 所有场景 |
| request       | 在一次http请求内有效 | Web场景  |
| session       | 在一个用户会话内有效 | Web场景  |
| globalSession | 在全局会话内有效     | Web场景  |



#### singleton

Springboot的注入默认范围是单例，在容器中通过对象引入配置注入和通过容器的getBean()方法返回的实例都是同一个bean。容器在启动的时候，自动实例化所有的singleton的bean并缓存于容器当中。

1、对bean提前的实例化操作，会及早发现一些潜在的配置的问题。

2、Bean以缓存的方式运行，当运行到需要使用该bean的时候，就不需要再去实例化了。加快了运行效率。



##### 线程安全问题

单例 bean 存在线程问题，主要是因为当多个线程操作同一个对象的时候，对这个对象的非静态成员变量的写操作会存在线程安全问题。

常见的有两种解决办法：

1. 在Bean对象中尽量避免定义可变的成员变量。
2. 在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在 ThreadLocal 中（推荐的一种方式）。



#### prototype

是指每次从容器中调用Bean时，都返回一个新的实例，即每次调用getBean()时，相当于执行new Bean()的操作。在默认情况下，容器在启动时不实例化prototype的Bean，容器将prototype的实例交给调用者之后便不在管理他的生命周期了。

对于有状态的Bean应该使用prototype，对于无状态的Bean则使用singleton



#### request

对应一个http请求和生命周期，当http请求调用作用域为request的bean的时候,Spring便会创建一个新的bean，在请求处理完成之后便及时销毁这个bean。



#### session

Session中所有http请求共享同一个请求的bean实例。Session结束后就销毁bean。



#### globalSession

与session大体相同，但仅在portlet应用中使用。Portlet规范定义了全局session的概念。请求的bean被组成所有portlet的自portlet所共享。如果不是在portlet这种应用下，globalSession则等价于session作用域。



### 对象声明周期



## 注解

### @Component

该注解作用于类，通过类路径扫描来自动侦测以及自动装配到Spring容器中（可以使用 `@ComponentScan` 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。



### @Bean

该注解作用于方法，通常是在标有该注解的方法中定义产生这个 Bean，`@Bean`告诉了Spring这是某个类的示例，当我需要用它的时候还给我。

`@Bean` 注解比 `Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册bean。比如当我们引用第三方库中的类需要装配到 `Spring`容器时，则只能通过 `@Bean`来实现。



### 总结

- Bean 容器找到配置文件中 Spring Bean 的定义。
- Bean 容器利用 Java Reflection API 创建一个Bean的实例。
- 如果涉及到一些属性值 利用 `set()`方法设置一些属性值。
- 如果 Bean 实现了 `BeanNameAware` 接口，调用 `setBeanName()`方法，传入Bean的名字。
- 如果 Bean 实现了 `BeanClassLoaderAware` 接口，调用 `setBeanClassLoader()`方法，传入 `ClassLoader`对象的实例。
- 与上面的类似，如果实现了其他 `*.Aware`接口，就调用相应的方法。
- 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessBeforeInitialization()` 方法
- 如果Bean实现了`InitializingBean`接口，执行`afterPropertiesSet()`方法。
- 如果 Bean 在配置文件中的定义包含 init-method 属性，执行指定的方法。
- 如果有和加载这个 Bean的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessAfterInitialization()` 方法
- 当要销毁 Bean 的时候，如果 Bean 实现了 `DisposableBean` 接口，执行 `destroy()` 方法。
- 当要销毁 Bean 的时候，如果 Bean 在配置文件中的定义包含 destroy-method 属性，执行指定的方法。



![Spring Bean 生命周期](../../Image/2022/07/220719-2.png)



## 接口

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



# Spring AOP

AOP(Aspect-Oriented Programming:面向切面编程)能够将那些与业务无关，**却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来**，便于**减少系统的重复代码**，**降低模块间的耦合度**，并**有利于未来的可拓展性和可维护性**。

**Spring AOP就是基于动态代理的**，如果要代理的对象，实现了某个接口，那么Spring AOP会使用**JDK Proxy**，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候Spring AOP会使用**Cglib** ，这时候Spring AOP会使用 **Cglib** 生成一个被代理对象的子类来作为代理，如下图所示：

![SpringAOPProcess](../../Image/2022/07/220719-1.png)



## 扩展

##### Spring AOP 和 AspectJ AOP 有什么区别？

**Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。** Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)。

Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ 相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，

如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择 AspectJ ，它比Spring AOP 快很多。



# Spring MVC

## 原理

### 核心执行流程

![SpringMVC运行原理](../../Image/2022/07/220719-3.png)



1. 客户端（浏览器）发送请求，直接请求到 `DispatcherServlet`。
2. `DispatcherServlet` 根据请求信息调用 `HandlerMapping`，解析请求对应的 `Handler`。
3. 解析到对应的 `Handler`（也就是我们平常说的 `Controller` 控制器）后，开始由 `HandlerAdapter` 适配器处理。
4. `HandlerAdapter` 会根据 `Handler `来调用真正的处理器开处理请求，并处理相应的业务逻辑。
5. 处理器处理完业务后，会返回一个 `ModelAndView` 对象，`Model` 是返回的数据对象，`View` 是个逻辑上的 `View`。
6. `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`。
7. `DispaterServlet` 把返回的 `Model` 传给 `View`（视图渲染）。
8. 把 `View` 返回给请求者（浏览器）



```java
o.s.w.servlet.FrameworkServlet#doGet
o.s.w.servlet.FrameworkServlet#processRequest
o.s.w.servlet.DispatcherServlet#doService
o.s.w.servlet.DispatcherServlet#doDispatch
o.s.w.servlet.DispatcherServlet#getHandler
o.s.w.servlet.handler.AbstractHandlerMethodMapping#getHandlerInternal
o.s.w.servlet.handler.AbstractHandlerMethodMapping#lookupHandlerMethod
o.s.w.servlet.HandlerAdapter#handle
o.s.w.servlet.mvc.method.annotation.RequestMappingHandlerAdapter#invokeHandlerMethod
o.s.w.servlet.mvc.method.annotation.ServletInvocableHandlerMethod#invokeAndHandle
o.s.w.method.support.InvocableHandlerMethod#invokeForRequest
o.s.w.method.support.InvocableHandlerMethod#getMethodArgumentValues
o.s.w.method.support.InvocableHandlerMethod#doInvoke
o.s.w.method.support.HandlerMethodReturnValueHandlerComposite#handleReturnValue
o.s.w.servlet.mvc.method.annotation.RequestMappingHandlerAdapter#getModelAndView
```







# Spring TX

## 编程式事务

编程式事务管理使用TransactionTemplate或者直接使用底层的PlatformTransactionManager。对于编程式事务管理，spring推荐使用TransactionTemplate。



## 声明式事务

声明式事务管理建立在AOP之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。



### 基于XML

基于XML的声明式事务。



### 基于注解

Spring在TransactionDefinition接口中规定了7种类型的事务传播行为，它们规定了事务方法和事务方法发生嵌套调用时事务如何进行传播：



#### 事务隔离级别

TransactionDefinition 接口中定义了五个表示隔离级别的常量。



| 类型                           | 描述                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| **ISOLATION_DEFAULT**          | 使用后端数据库默认的隔离级别，Mysql 默认采用的 REPEATABLE_READ隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别。 |
| **ISOLATION_READ_UNCOMMITTED** | 最低的隔离级别，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读** |
| **ISOLATION_READ_COMMITTED**   | 允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生** |
| **ISOLATION_REPEATABLE_READ**  | 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生。** |
| **ISOLATION_SERIALIZABLE**     | 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。但是这将严重影响程序的性能。通常情况下也不会用到该级别。 |



#### 事务传播行为

TransactionDefinition 接口中定义了七个表示事务传播行为的常量。



##### 支持当前事务

| 类型                      | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| **PROPAGATION_REQUIRED**  | 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。 |
| **PROPAGATION_SUPPORTS**  | 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。 |
| **PROPAGATION_MANDATORY** | 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性） |



##### 不支持当前事务

| 类型                          | 描述                                                   |
| ----------------------------- | ------------------------------------------------------ |
| **PROPAGATION_REQUIRES_NEW**  | 创建一个新的事务，如果当前存在事务，则把当前事务挂起。 |
| **PROPAGATION_NOT_SUPPORTED** | 以非事务方式运行，如果当前存在事务，则把当前事务挂起。 |
| **PROPAGATION_NEVER**         | 以非事务方式运行，如果当前存在事务，则抛出异常。       |



##### 其它

| 类型                   | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| **PROPAGATION_NESTED** | 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于 **PROPAGATION_REQUIRED**。 |



#### @Transactional

当`@Transactional`注解作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。如果类或者方法加了这个注解，那么这个类里面的方法抛出异常，就会回滚，数据库里面的数据也会回滚。

在`@Transactional`注解中如果不配置`rollbackFor`属性,那么事物只会在遇到`RuntimeException`的时候才会回滚,加上`rollbackFor=Exception.class`,可以让事物在遇到非运行时异常时也回滚。



##### 源码解析

> org.springframework.transaction.annotation.Transactional

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {

	/**
	 * 
	 */
	@AliasFor("transactionManager")
	String value() default "";

	/**
	 * 
	 */
	@AliasFor("value")
	String transactionManager() default "";

	/**
	 * 事务传播行为
	 */
	Propagation propagation() default Propagation.REQUIRED;

	/**
	 * 事务隔离级别
	 */
	Isolation isolation() default Isolation.DEFAULT;

	/**
	 * 
	 */
	int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;

	/**
	 * 
	 */
	boolean readOnly() default false;

	/**
	 * 
	 */
	Class<? extends Throwable>[] rollbackFor() default {};

	/**
	 * 
	 */
	String[] rollbackForClassName() default {};

	/**
	 * 
	 */
	Class<? extends Throwable>[] noRollbackFor() default {};

	/**
	 * 
	 */
	String[] noRollbackForClassName() default {};
```

