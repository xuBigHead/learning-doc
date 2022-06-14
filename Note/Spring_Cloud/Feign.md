# 目录
[TOC]

# Feign 概述
**1、概述**

`Feign`是一个声明性web服务客户端。通过定义一个服务接口并在上面添加注解的方式来使编写web服务客户端变得更容易。`Feign`支持可插拔编码器和解码器，且可以与注册中心（如`Eureka`）和`Ribbon`组合使用来支持负载均衡。

`OpenFeign`是`Spring Cloud`对`Feign`进行的封装，使其支持`SpringMVC`标准注解和`HttpMessageConverters`。



**2、Feign能干什么**

Feign旨在使编写Java Http客户端变得更容易。

前面在使用Ribbon+RestTemplate时，利用RestTemplate对http请求的封装处理，形成了一套模版化的调用方法。但在实际开发中，由于对服务依赖的调用可能不止一处，`往往一个接口可能被多处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用`。所以，Feign在此基础上做了进一步封装，由它来帮助我们定义和实现依赖服务接口的定义。在Feign的实现下，`我们只需要创建一个接口并使用注解的方式来配置它（以前是Dao接口上面标注Mapper注解，现在是一个微服务接口上标注一个Feign注解即可）`，即可完成对服务提供方的的接口绑定，简化了使用Spring Cloud Ribbon时，自动封装服务调用客户端的开发量。



**3、Feign集成了Ribbon**

利用Ribbon维护了Payment的服务列表信息，并且通过轮询实现了客户端的负载均衡。而与Ribbon不同的是，`通过Feign只需要定义服务绑定接口且以声明式的方法`，优雅而简单的实现了服务的调用。



**4、Feign和OpenFeign的区别**

| Feign                                                        | OpenFeign                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Feign是SpringCloud组件中一个轻量级RESTful的HTTP服务客户端，Feign内置了Ribbon,用来做客户端负载均衡，去调用服务注册中心的服务。Feign的使用方式是：使用Feign的注解定义接口，调用这个接口，就可以调用服务注册中心的服务 | OpenFeign 是SpringCloud在Feign的基础上支持了SpringMVC的注解，如@RequestMapping等。OpenFeign 的@FeignClient可以解析SpringMVC的@RequestMapping注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务。 |

## 原理
- 启动时，程序会进行包扫描，扫描所有包下所有@FeignClient注解的类，并将这些类注入到spring的IOC容器中。当定义的Feign中的接口被调用时，通过JDK的动态代理来生成RequestTemplate。
- RequestTemplate中包含请求的所有信息，如请求参数，请求URL等。
- RequestTemplate生产Request，然后将Request交给client处理，这个client默认是JDK的HTTPUrlConnection，也可以是OKhttp、Apache的HTTPClient等。
- 最后client封装成LoadBaLanceClient，结合ribbon负载均衡地发起调用。

# Feign 示例
引入OpenFeign的maven依赖，当前文档以openfeign的2.2.2.RELEASE版本为基准。

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
    <version>2.2.2.RELEASE</version>
</dependency>
```

在SpringBoot启动器上添加`@EnableFeignClients`注解，并配置要扫描的Feign客户端的包。
```java
@EnableFeignClients("com.demo.feign")
@SpringBootApplication
public class ServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServiceApplication.class, args);
    }
}
```

创建Feign接口和实现该接口的控制器
```java
@FeignClient(
        value = "base-service",
)
public interface IProviderClient {
    String API_PREFIX = "/client";
    String GET_USER_BY_ID = API_PREFIX + "/get-user-by-id";
    
    /**
     * 通过feign调用get请求获取用户信息
     *
     * @param id id
     * @return user
     */
    @GetMapping(GET_USER_BY_ID)
    R<User> getUserById(@RequestParam("id") Long id);
}
```

实现远程调用需要执行的方法。

```java
@RestController
public class ProviderClient implements IProviderClient {
    @Override
    public R<User> getUserById(Long id) {
        return R.data(User.builder().id(id).realName("mangmang.xu").build());
    }
}
```

通过引入该接口并调用其方法即可获取user信息。

```java
@RestController
@AllArgsConstructor
public class FeignConsumerController {
    private final IProviderClient providerClient;
    @GetMapping("/get-user-by-id")
    public R<User> detail(@RequestParam("id") Long id) {
        return providerClient.getUserById(id);
    }
}
```
# Feign 注解
## @EnableFeignClients
使用`@EnableFeignClients`表示当前服务开启Feign客户端功能。
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(FeignClientsRegistrar.class)
public @interface EnableFeignClients {
	String[] value() default {};
	String[] basePackages() default {};
	Class<?>[] basePackageClasses() default {};
	Class<?>[] defaultConfiguration() default {};
	Class<?>[] clients() default {};
}
```

## @FeignClient
使用`@FeignClient`注解标注接口来表名一个Feign客户端。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FeignClient {
	@AliasFor("name")
	String value() default "";
	@Deprecated
	String serviceId() default "";
	String contextId() default "";
	@AliasFor("value")
	String name() default "";
	String qualifier() default "";
	String url() default "";
	boolean decode404() default false;
	Class<?>[] configuration() default {};
	Class<?> fallback() default void.class;
	Class<?> fallbackFactory() default void.class;
	String path() default "";
	boolean primary() default true;
}
```

| 属性名          | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| value           | 调用服务名称，和name属性相同                                 |
| serviceId       | 服务id，作用和name属性相同                                   |
| contextId       |                                                              |
| name            | 调用服务名称，和value属性相同                                |
| qualifier       |                                                              |
| url             | 全路径地址或hostname，http或https可选                        |
| decode404       | 配置响应状态码为404时是否应该抛出FeignExceptions             |
| configuration   | 自定义当前feign client的一些配置，参考FeignClientsConfiguration |
| fallback        | 熔断机制，调用失败时，走的一些回退方法，可以用来抛出异常或给出默认返回数据。底层依赖hystrix，启动类要加上@EnableHystrix。 |
| fallbackFactory |                                                              |
| path            | 给所有方法的requestMapping前加上前缀，类似与controller类上的requestMapping |
| primary         |                                                              |



# Feign 异常
## FeignException
Feign通过http请求调用远程方法失败时会跑出FeignException



## 异常相关源码

`feign.SynchronousMethodHandler.executeAndDecode`

```java
    Object executeAndDecode(RequestTemplate template, Options options) throws Throwable {
        // ...
        if (response.status() >= 200 && response.status() < 300) {
            // http状态码code大于等于200，小于300时正常返回
        } else if (decode404 && response.status() == 404 && void.class != metadata.returnType()) {
            // http状态码code为404
        } else {
            // 远程调用失败，抛出FeignException或自定义的异常
            throw errorDecoder.decode(metadata.configKey(), response);
        }
        // ...
    }
```

`feign.codec.ErrorDecoder.Default.decode`
```java
    public Exception decode(String methodKey, Response response) {
        FeignException exception = errorStatus(methodKey, response);
        // ...
        return exception;
    }
```



## 自定义返回异常

- 创建抛出自定义异常类
```java
public class FeignErrorDecoder implements ErrorDecoder {
    @Override
    @SneakyThrows
    public Exception decode(String methodKey, Response response) {
        log.info("自定义Feign调用异常信息");
        if (response.status() >= HttpConstant.STATUS_CODE_500 && response.status() <= HttpConstant.STATUS_CODE_599) {
            log.error("Feign调用异常，异常原因[{}]", response.reason());
            byte[] body = response.body() != null ? Util.toByteArray(response.body().asInputStream()) : new byte[]{};
            return new ServiceException("Feign调用异常，异常原因：" + new String(body));
        }
        // 执行默认的Feign异常处理流程
        return new Default().decode(methodKey, response);
    }
}
```

- 通过`@FeignClient`注解指定该抛出异常类
```java
@FeignClient(
        value = "base-service",
        configuration = FeignErrorDecoder.class
)
public interface IProviderClient {
    // ...
}
```

## 参考资料
- [ ] [Feign自定义ErrorDecoder错误时返回统一结构](https://blog.csdn.net/new9xgh/article/details/107934862)



# Feign 熔断
Feign熔断Fallback主要是用来解决依赖的服务不可用或者调用服务失败或超时，使用默认的返回值。因为Fallback是通过Hystrix实现的， 所以需要开启Hystrix。

- 开启开启hystrix配置
```yaml
feign:
  hystrix:
    # 开启hystrix实现熔断功能
    enabled: true
```

## Fallback实现类

- 创建Fallback类实现Feign客户端接口的方法

远程调用执行方法失败时，通过Hystrix来调用fallback类中的该方法的实现，避免引发服务雪崩等问题。
```java
@Component
public class ProviderClientFallback implements IProviderClient{
    @Override
    public R<User> getUserById(Long id) {
        return R.fail("根据id获取用户信息失败");
    }
}
```

- 配置Fallback类

通过`@FeignClient`注解的fallback属性配置Fallback类   
```java
@FeignClient(
        value = "base-service",
        fallback = ProviderClientFallback.class
)
public interface IProviderClient {
    // ...
}
```

## Fallback工厂
上面的实现方式简单，但是获取不到HTTP请求错误状态码和信息 ，这时就可以使用工厂模式来实现Fallback。

- 创建FallbackFactory工厂类
```java
@Slf4j
@Component
@AllArgsConstructor
public class UserQueryClientFactory implements FallbackFactory<IUserQueryClient> {
    private final UserQueryClientFallback userQueryClientFallback;

    @Override
    public IUserQueryClient create(Throwable cause) {
        log.error("feign请求调用异常，异常信息为[{}]", cause.getMessage());
        cause.printStackTrace();
        return userQueryClientFallback;
    }
}
```

- 将创建的FallbackFactory工厂类配置到`@FeignClient`注解的`fallbackFactory`属性中

```java
@FeignClient(
        value = "base-service",
        fallbackFactory = UserQueryClientFactory.class
)
public interface IProviderClient {
    // ...
}
```

## Hystrix加载Fallback类

在`@FeignClient`注解中同时配置`fallback`属性和`fallbackFactory`属性时，只有fallback属性配置的Fallback类会被加载。
```java

class HystrixTargeter implements Targeter {
    @Override
    public <T> T target(FeignClientFactoryBean factory, Feign.Builder feign,
                        FeignContext context, Target.HardCodedTarget<T> target) {
        // ...
        Class<?> fallback = factory.getFallback();
        if (fallback != void.class) {
            // 如果配置了@FeignClient注解的fallback属性，则不会再加载fallbackFactory属性的配置
            return targetWithFallback(name, context, target, builder, fallback);
        }
        Class<?> fallbackFactory = factory.getFallbackFactory();
        if (fallbackFactory != void.class) {
            return targetWithFallbackFactory(name, context, target, builder,
                fallbackFactory);
        }

        return feign.target(target);
    }
}
```

# Feign 日志

## 默认日志
Feign的Level日志级别配置默认是:NONE，和log日志级别不同。

- 首先配置如下的config类
```java
@Configuration
public class FeignConfiguration {
    @Bean
    Logger.Level feignLevel() {
        return Logger.Level.FULL;
    }
}
```

- 然后配置feign客户端的日志级别为debug
```yaml
logging:
  level:
    com.demo.feign: debug
```

## 日志输出相关源码

日志级别枚举类 `feign.Logger.Level`
```java
public enum Level {
    NONE,
    BASIC,
    HEADERS,
    FULL
}
```

| 属性名  | 说明                                                 |
| ------- | ---------------------------------------------------- |
| NONE    | 不输出日志                                           |
| BASIC   | 只有请求方法、URL、响应状态代码、执行时间            |
| HEADERS | 基本信息以及请求和响应头                             |
| FULL    | 请求和响应 的heads、body、metadata，建议使用这个级别 |

日志输出操作

`feign.SynchronousMethodHandler.executeAndDecode`
```java
    Object executeAndDecode(RequestTemplate template, Options options) throws Throwable {
        // ...
        // 判断输出日志级别是否大于NONE，然后输出请求日志信息
        if (logLevel != Logger.Level.NONE) {
            logger.logRequest(metadata.configKey(), logLevel, request);
        }
        // ...
        // 判断输出日志级别是否大于NONE，然后输出响应日志信息
        if (logLevel != Logger.Level.NONE) {
            response = logger.logAndRebufferResponse(metadata.configKey(), 
                logLevel, response, elapsedTime);
        }
    }
```

默认的Feign日志输出类

```java
public class Slf4jLogger extends feign.Logger {    
    private final org.slf4j.Logger logger;
    // ...
    @Override
    protected void logRequest(String configKey, Level logLevel, Request request) {
        if (logger.isDebugEnabled()) {
            super.logRequest(configKey, logLevel, request);
        }
    }

    @Override
    protected Response logAndRebufferResponse(String configKey, Level logLevel, Response response, long elapsedTime)
        throws IOException {
        if (logger.isDebugEnabled()) {
            return super.logAndRebufferResponse(configKey, logLevel, response, elapsedTime);
        }
        return response;
    }

    @Override
    protected void log(String configKey, String format, Object... args) {
        if (logger.isDebugEnabled()) {
            logger.debug(String.format(methodTag(configKey) + format, args));
        }
    }
}
```

默认的Feign日志工厂类
```java
public class DefaultFeignLoggerFactory implements FeignLoggerFactory {
	private Logger logger;
	public DefaultFeignLoggerFactory(Logger logger) {
		this.logger = logger;
	}
	@Override
	public Logger create(Class<?> type) {
		return this.logger != null ? this.logger : new Slf4jLogger(type);
	}
}
```

`org.springframework.cloud.openfeign.FeignClientsConfiguration`配置类中有如下配置，如果有自定义的`FeignLoggerFactory`，就不加载默认的`FeignLoggerFactory`。
```java
	@Bean
	@ConditionalOnMissingBean(FeignLoggerFactory.class)
	public FeignLoggerFactory feignLoggerFactory() {
		return new DefaultFeignLoggerFactory(this.logger);
	}
```

## 自定义日志

Feign的日志是Debug级别的，不符合线上输出info级别日志的要求，openfeign对日志输出做了扩展，可以通过实现`FeignLoggerFactory`接口和继承`Logger`类来自定义日志输出。

- 继承`Logger`类
```java
public class FeignInfoLogger extends Logger {
    private final org.slf4j.Logger logger;

    FeignInfoLogger(org.slf4j.Logger logger) {
        this.logger = logger;
    }

    @Override
    protected void logRequest(String configKey, Level logLevel, Request request) {
        logger.info("输出Feign接口[{}]请求日志信息", configKey);
        if (this.logger.isDebugEnabled() || this.logger.isInfoEnabled()) {
            super.logRequest(configKey, logLevel, request);
        }
    }

    @Override
    protected Response logAndRebufferResponse(String configKey, Level logLevel, Response response, long elapsedTime) throws IOException {
        logger.info("输出Feign接口[{}]响应日志信息", configKey);
        return this.logger.isDebugEnabled() || this.logger.isInfoEnabled() ? super.logAndRebufferResponse(configKey, logLevel, response, elapsedTime) : response;
    }

    @Override
    protected void log(String configKey, String format, Object... args) {
        // 当日志级别设置为info级别时，输出info级别的日志信息
        if (this.logger.isInfoEnabled()) {
            this.logger.info(String.format(methodTag(configKey) + format, args));
        }
    }
}
```

- 实现`FeignLoggerFactory`接口
```java
public class FeignInfoLoggerFactory implements FeignLoggerFactory {
    @Override
    public Logger create(Class<?> type) {
        return new FeignInfoLogger(LoggerFactory.getLogger(type));
    }
}
```

- 将自定义的Feign日志工厂类注入到容器中
```java
@Configuration
public class FeignConfiguration {
    @Bean
    FeignLoggerFactory feignLoggerFactory() {
        return new FeignInfoLoggerFactory();
    }
}
```

# Feign 配置
## Feign配置方式
- 通过yaml配置文件进行配置，该方式优先级最低；
- 通过`@FeignClient`注解的`configuration`属性进行配置；
- 通过`@EnableFeignClients`注解的`defaultConfiguration`属性进行配置，该方式优先级最高；

## 超时配置

Hystrix在最外层，然后再到Ribbon，最后里面的是http请求。所以说。Hystrix的熔断时间必须大于Ribbon的 ( ConnectTimeout + ReadTimeout )。而如果Ribbon开启了重试机制，还需要乘以对应的重试次数，保证在Ribbon里的请求还没结束时，Hystrix的熔断时间不会超时。

### Ribbon

```yaml
#设置feign客户端超时时间（openfeign默认支持ribbon）
ribbon:
  #指的是建立连接所用的时间，适用于网络状况正常的情况下，两端连接所用的时间
  ReadTimeout: 10000
  #指的是建立连接后从服务器读取到可用资源所用的时间
  ConnectTimeout: 3000000
```

### Hystrix



## Http客户端配置
`Feign`底层默认是使用jdk中的`HttpURLConnection`发送HTTP请求，`SpringCloud`从`Brixton.SR5`版本开始`feign`提供了`OKhttp`和Apache的`HTTPClient`来发送请求，具体配置如下：



### OKhttp配置
- 引入OKhttp的依赖
```xml
    <dependency>
        <groupId>io.github.openfeign</groupId>
        <artifactId>feign-okhttp</artifactId>
    </dependency>
```

- 添加yaml配置
```yaml
feign:
  okhttp:
    # 开启OKhttp客户端
    enabled: true
```

- 添加OKhttp配置类
```java
@Configuration
@ConditionalOnClass(Feign.class)
@AutoConfigureBefore(FeignAutoConfiguration.class)
public class FeignClientOkHttpConfiguration {
    @Bean
    public OkHttpClient okHttpClient() {
        return new OkHttpClient.Builder()
            // 连接超时
            .connectTimeout(20, TimeUnit.SECONDS)
            // 响应超时
            .readTimeout(20, TimeUnit.SECONDS)
            // 写超时
            .writeTimeout(20, TimeUnit.SECONDS)
            // 是否自动重连
            .retryOnConnectionFailure(true)
            // 连接池
            .connectionPool(new ConnectionPool())
            .build();
    }
}
```



### HttpClient配置

- 引入HttpClient的依赖
```xml
    <dependency>
        <groupId>org.apache.httpcomponents</groupId>
        <artifactId>httpclient</artifactId>
    </dependency>
    <dependency>
        <groupId>io.github.openfeign</groupId>
        <artifactId>feign-httpclient</artifactId>
    </dependency>
```

- 添加yaml配置

```yaml
feign:
  httpclient:
    # 开启HttpClient
    enabled: true
    # 最大连接数，默认：200
    max-connections: 200
    # 最大路由，默认：50
    max-connections-per-route: 50
    # 连接超时，默认：2000/毫秒
    connection-timeout: 2000
    # 生存时间，默认：900L
    time-to-live: 900
    # 响应超时的时间单位，默认：TimeUnit.SECONDS
    time-to-live-unit: seconds
```

**HttpClient相关配置类如下**

FeignHttpClientProperties负责配置HttpClient相关属性。
```java
@ConfigurationProperties(prefix = "feign.httpclient")
public class FeignHttpClientProperties {
    public static final boolean DEFAULT_DISABLE_SSL_VALIDATION = false;
    public static final int DEFAULT_MAX_CONNECTIONS = 200;
    public static final int DEFAULT_MAX_CONNECTIONS_PER_ROUTE = 50;
    public static final long DEFAULT_TIME_TO_LIVE = 900L;
    public static final TimeUnit DEFAULT_TIME_TO_LIVE_UNIT;
    public static final boolean DEFAULT_FOLLOW_REDIRECTS = true;
    public static final int DEFAULT_CONNECTION_TIMEOUT = 2000;
    public static final int DEFAULT_CONNECTION_TIMER_REPEAT = 3000;
    private boolean disableSslValidation = false;
    private int maxConnections = 200;
    private int maxConnectionsPerRoute = 50;
    private long timeToLive = 900L;
    private TimeUnit timeToLiveUnit;
    private boolean followRedirects;
    private int connectionTimeout;
    private int connectionTimerRepeat;
}
```

HttpClientFeignLoadBalancedConfiguration负责将HttpClient客户端注入到Feign中。
```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(ApacheHttpClient.class)
@ConditionalOnProperty(value = "feign.httpclient.enabled", matchIfMissing = true)
@Import(HttpClientFeignConfiguration.class)
class HttpClientFeignLoadBalancedConfiguration {
	@Bean
	@ConditionalOnMissingBean(Client.class)
	public Client feignClient(CachingSpringLoadBalancerFactory cachingFactory,
			SpringClientFactory clientFactory, HttpClient httpClient) {
		ApacheHttpClient delegate = new ApacheHttpClient(httpClient);
		return new LoadBalancerFeignClient(delegate, cachingFactory, clientFactory);
	}
}
```

### 参考资料
- [ ] [[享学Feign] 九、Feign + OkHttp和Feign + Apache HttpClient哪个更香？](https://blog.csdn.net/f641385712/article/details/104305106)
- [x] [Feign 默认 Client 替换](https://blog.csdn.net/wo18237095579/article/details/83377938)
- [ ] [Feign、httpclient、OkHttp3 结合使用](https://www.cnblogs.com/crazymakercircle/p/11968479.html)

## 请求和响应数据压缩配置
```yaml
feign:
  compression:
    request:
      # feign请求压缩配置
      enabled: true
      # feign压缩请求支持的格式
      mime-types: text/xml,application/xml,application/json
      # feign压缩请求最小的大小
      min-request-size: 2048
    response:
      # feign响应压缩配置
      enabled: true
```

# Feign 执行流程
- org.springframework.cloud.openfeign.FeignClientFactoryBean.getTarget(服务启动时)
- feign.Feign.Builder.build(服务启动时)
- feign.ReflectiveFeign.FeignInvocationHandler.invoke(调用Feign客户端接口时)
- feign.SynchronousMethodHandler.invoke(调用Feign客户端接口时)

## 注册Feign客户端

## 发送请求Request

## 接收响应Response

## 参考资料
- [ ] [Spring Cloud feign客户端执行流程概述](https://blog.csdn.net/andy_zhang2007/article/details/86720165)

# Feign 实践

## Get请求传递对象
feign不支持GET请求直接传递POJO对象的，目前解决方法如下：
- 把POJO拆散城一个一个单独的属性放在方法参数中；
- 把方法参数编程Map传递；
- 使用GET传递@RequestBody，但此方式违反restful风格；
- 通过feign的拦截器来实现（推荐）；

```java
@Component
@AllArgsConstructor
public class FeignGetRequestInterceptor implements RequestInterceptor {
    private final ObjectMapper objectMapper;

    @Override
    @SneakyThrows
    public void apply(RequestTemplate template) {
        if (Request.HttpMethod.GET.toString().equalsIgnoreCase(template.method())
            && template.body() != null) {
            //feign 不支持GET方法传输POJO 转换成json，再换成query
            Map<String, Object> map =
                objectMapper.readValue(template.body(),
                    new TypeReference<Map<String, Object>>() {});
            Map<String, Collection<String>> queries = new HashMap<>(map.size());
            for (String key : map.keySet()) {
                Object value = map.get(key);
                    queries.put(key, value instanceof Collection
                        ? ((Collection<?>) value).stream().map(String::valueOf).collect(Collectors.toList())
                        : Lists.newArrayList(value.toString()));
            }
            template.body(StringPool.EMPTY);
            template.queries(queries);
        }
    }
}
```

### 参考资料
- [Feign发送Get请求时，采用POJO对象传递参数的最终解决方案](https://blog.csdn.net/f641385712/article/details/82431502)



# 参考资料

## 官方
- [Spring Cloud OpenFeign官网](https://spring.io/projects/spring-cloud-openfeign)
- [OpenFeign/feign-Github地址](https://github.com/OpenFeign/feign)

## 博客
- [Spring Cloud Feign 异常处理](https://my.oschina.net/xiaominmin/blog/2986631)
- [Spring Cloud Feign](https://blog.csdn.net/sun_shaoping/category_8057928.html)
- [Spring Cloud Feign 调用过程分析](https://segmentfault.com/a/1190000039230324)
- [Feign Client 配置](https://xli1224.github.io/2017/09/22/configure-feign/) 
- [微服务实战SpringCloud之Feign源码分析](https://www.jianshu.com/p/ce6631e8c762)

- [AI全栈程序猿 ](https://www.jianshu.com/u/88e407ec7d34)

# 未验证内容

由于开启GZIP压缩之后，Feign之间的调用数据通过二进制协议进行传输，返回值需要修改为ResponseEntity<byte[]>才可以正常显示，否则会导致服务之间的调用乱码。
```java
@PostMapping("/order/{productId}")
ResponseEntity<byte[]> addCart(@PathVariable("productId") Long productId);
```

---
