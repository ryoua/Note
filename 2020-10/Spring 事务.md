有关事务的前置知识就不说了, 感兴趣可以看MySQL中事务的概念

# Spring事务

在了解Spring的事务管理之前, 我们先来了解一下常见事务场景中的角色

* Resource Manager. 简称RM, 负责存储并管理系统数据资源的状态, 比如DB, 消息队列都是RM
* Transaction Processing Monitor. 简称TP Monitor, 职责主要是在分布式事务场景中协调多个RM的事务处理.
* TransactionManager. 简称TM, 可以认为是TPM中的核心模块, TPM包括了多个TM. TM负责多个RM之间的事务处理的协调工作, 还提供上下文传播, 事务界定等功能
* Application. 即我们的应用

以上就是一个常见事务处理中的一些角色, 但是并不是每个事务场景都会出现上面的角色, 事务主要分为全局事务和局部事务. 

在全局事务中, 如果整个处理过程中有多个RM参与, 那么就需要TPM来进行协调, TPM一般通过2PC来保证事务的ACID特性

在局部事务中, 往往只有一个RM参与, 这个时候因为只有一个RM, 所以没必要引入TPM来增加额外的复杂度, 这个时候只需要让应用直接和RM打交道即可, 这种情况下就需要RM内置事务支持来实现我们需要的功能

在了解了事务中的角色后, 我们来看Spring中的事务具体原理, 相信大家在日常的开发中应该都多多少少使用过事务, 所以这里我先不讲事务的使用, 先看实现, 最后在了解了实现后, 再来看事务的使用, 就会理解的更深刻

在Spring的事务模块中, 最核心的接口就是PlatformTransactionManager, 它的主要作用就是为应用提供一个事务界定的同一方式, 其代码如下: 

```JAVA
public interface PlatformTransactionManager extends TransactionManager {
	TransactionStatus getTransaction(TransactionDefinition definition)
			throws TransactionException;

	void commit(TransactionStatus status) throws TransactionException;

	void rollback(TransactionStatus status) throws TransactionException;
}
```

可以看到接口很简单, 只有三个方法, 其中最主要的就是commit和rollback方法, 也就是提交和回滚. PlatformTransactionManager就像是一个战略蓝图, 给我们抽象出三个通用的方法, 具体事务如何实现, 统统交给底下的实现类去实现.

Spring的事务框架针对不同的数据访问形式提供了相应的实现类, 在这里我们主要以针对JDBC数据访问形式的事务管理做讲解, 在我们的日常开发中, 一般会将事务放在Service层而非Dao层.  这样可以避免相互之间的影响, 一般的Service对象需要在同一个业务方法中调用多个数据访问对象, 但是由于JDBC的事务控制是由同一个java.sql.Connection来完成的, 所以要保证两个Dao数据访问方法处于一个事务中, 我们就得保证他们用的是同一个Connection. 但是如何才能保证使用的是同一个Connection呢

###### 普通实现

最简单地, 我们可以把Connection当做参数传入方法中即可, 但是这样有一个很大的问题, 我们无法摆脱实现的依赖, 比如我们是JDBC访问, 我们需要Connection的依赖, 如果是Hibernate的话, 我们又得声明对Session的依赖. 显然, 这样是不可行的

###### 线程绑定实现

Spring采用了线程绑定去实现, 当我们在事务开始时, 会先取得一个Connection, 然后把Connection绑定到当前的调用线程, 之后数据访问对象在使用Connection进行数据访问的时候就可以在当前事务对应的线程中获取.

上面的实现在Spring中对应了ResourceTransactionManager接口, 代码如下

```java
public interface ResourceTransactionManager extends PlatformTransactionManager {
	Object getResourceFactory();
}

```

这个接口只有一个方法getResourceFactory, 作用就是用来获取一个Resource工厂, 也就是存放Connection的地方.

了解了PlatformTransactionManager之后, 我们需要了解一下另外两个接口, 具体关系如下:

![image](http://tvax3.sinaimg.cn/large/006A5NwJgy1gk0pz4sog9j30pk0fe0xh.jpg)

##### TransactionDefinition

TransactionDefinition主要定义了有哪些事务属性可以指定, 包括:

* 事务的隔离级别
* 事务的传播行为
* 事务的超时时间
* 事务是否为只读事务

其中隔离级别有五种, 除了DEFAULT即采用数据库默认的隔离级别之外, 其他的都和数据库的隔离级别一一对应

而传播行为来说, 主要有以下几种

* required: 如果当前存在一个事务, 则加入当前事务, 如果不存在, 则创建一个新的事务(默认)
* supports: 如果当前存在一个事务, 则加入当前事务, 如果不存在, 则直接执行
* mandatory: 强制要求当前存在一个事务, 如果不存在则直接抛出异常
* required_new: 不管当前是否存在事务, 都创建新的事务
* not_supported: 不支持当前事务, 在没有事务的情况下执行
* never: 永远不需要当前存在事务, 如果存在则抛出异常
* nested: 如果当前存在事务, 则在当前事务的一个嵌套事务中执行

不过TransactionDefinition只是一个接口定义, 要为PlatformTransactionManager创建事务提供信息, 我们需要一个实现类提供支持, TransactionDefinition的实现类大概可以分为两类: 编程式和声明式, 这两个区别我们后面再说, 







### 事务类型

##### 编程式事务

##### 声明式事务

