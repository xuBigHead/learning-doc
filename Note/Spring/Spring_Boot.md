# 目录

[TOC]

# 概述

Spring Boot是Spring开源组织下的子项目，是Spring组件一站式解决方案，主要是简化了使用Spring的难度，简省了繁重的配置，提供了各种启动器，开发者能快速上手。

## SpringBoot特点

## starter

# 配置

# 常用注解

## @ConditionalOnProperty

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
@Documented
@Conditional({OnPropertyCondition.class})
public @interface ConditionalOnProperty {
    //数组，获取对应property名称的值，与name不可同时使用
    String[] value() default {}; 
    //property名称的前缀，可有可无
    String prefix() default "";
 	//数组，property完整名称或部分名称（可与prefix组合使用，组成完整的property名称），与value不可同时使用
    String[] name() default {};
 	//可与name组合使用，比较获取到的属性值与havingValue给定的值是否相同，相同才加载配置
    String havingValue() default "";
 	//缺少该property时是否可以加载。如果为true，没有该property也会正常加载；反之报错
    boolean matchIfMissing() default false;
 	//是否可以松散匹配，至今不知道怎么使用的
    boolean relaxedNames() default true;
}
```



***example***

```java
@Bean
@ConditionalOnProperty(value = "sample.zipkin.enabled", havingValue = "false")
public Reporter<Span> spanReporter() {
    return Reporter.CONSOLE;
}
```



# 集成MyBatis

# 集成MyBatis-Plus

# 集成MongoDb

# 集成Redis

# 集成日志





- [SpringBoot中 Jackson 日期的时区和日期格式问题](https://blog.csdn.net/jianxia801/article/details/89741073)

