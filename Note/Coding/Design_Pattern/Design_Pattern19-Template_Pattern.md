# 设计模式19-模板方法模式（Template）
## 概述
> Define the skeleton of an algorithm in an operation, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps of an algorithm without changing the algorithm’s structure.

定义一个操作中的算法的框架， 而将一些步骤延迟到子类中。 使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

## 简单实现

## 基于JDK的实现

## 基于IOC容器的实现

## 源码解析

## 优缺点
### 优点
- 封装不变部分，扩展可变部分

把认为是不变部分的算法封装到父类实现，而可变部分的则可以通过继承来继续扩展。

- 提取公共部分代码，便于维护

- 行为由父类控制，子类实现

基本方法是由子类实现的，因此子类可以通过扩展的方式增加相应的功能，符合开闭原则。

### 缺点
子类执行的结果影响了父类的结果，在复杂项目中，会带来阅读上的难度。

可能引起子类泛滥和为了继承而继承的问题

## 使用场景

## 参考资料
- [Java设计模式之（十三）——模板方法模式 ](https://www.cnblogs.com/ysocean/p/15631116.html)