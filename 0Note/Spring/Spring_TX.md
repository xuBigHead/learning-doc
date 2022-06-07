# table of contents
[TOC]

# 事务
Spring在TransactionDefinition接口中规定了7种类型的事务传播行为，它们规定了事务方法和事务方
法发生嵌套调用时事务如何进行传播：

## @Transactional注解

PROPAGATION_REQUIRED
如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选
择。
PROPAGATION_SUPPORTS
支持当前事务，如果当前没有事务，就以非事务方式执行。
PROPAGATION_MANDATORY
使用当前的事务，如果当前没有事务，就抛出异常。
PROPAGATION_REQUIRES_NEW
新建事务，如果当前存在事务，把当前事务挂起。
PROPAGATION_NOT_SUPPORTED
以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
PROPAGATION_NEVER
以非事务方式执行，如果当前存在事务，则抛出异常。
PROPAGATION_NESTED
如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类
似的操作。