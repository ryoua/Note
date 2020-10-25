# Spring Ioc

**Ioc**的基本概念

IOC和中文名叫依赖注入，也叫控制反转。IOC的核心思想就是让别人为你服务，在传统的开发中，当我们需要一个对象时，通常是自己手动去new，但是通过ioc，可以将这个过程翻转过来，ioc来主动给我们对象，而我们只需要给定注入的参数

注入的方式

- 构造方法注入：对象构造完成后即可使用，缺点是当依赖对象很多时，构造方法的参数会变得很多，而且多个构造方法可能造成维护上的困难
- setter方法注入：描述性和维护性要好一点，但是缺点是构造完还不能立即使用
- 接口注入：侵入性太强，已经被废弃

## Ioc Service Provider

我们可以通过Ioc方式声明相应的依赖，但是仅仅声明依赖肯定是不行的，最终肯定是需要通过某种服务或者角色来将这些相互以来的对象绑定到一起，而Ioc Service Provider就对应着这个角色

Ioc Service Provider是一个抽象出来的概念，可以是一段代码，就比如new一个对象，然后通过构造方法的形式去注入两个依赖对象，也可以是一个Ioc容器，比如Spring的BeanFactory

**Ioc Service Provider**的职责

- 业务对象的构建管理：将对象的构建逻辑从客户端对象中剥离出来，以免污染客户端的代码
- 业务对象间的依赖绑定：通过之前的构建和管理的所有业务对象，以及各个对象之间的依赖关系，将对象所依赖的对象注入绑定，从而保证每个业务对象在使用的时候，可以处于就绪的状态

**Ioc Service Provider**的使用方式

- 文本文件
- XML
- 代码
- ......

在Spring当中，提供了两种Ioc Service Provider，分别是

- BeanFactory：基础的Ioc容器，提供了基本的Ioc支持，默认采用延迟初始化策略，在资源有限，功能要求不是很严格的场景，BeanFactory是比较合适的Ioc容器选择
- ApplicationContext： 在BeanFactory的基础上构建，是比较高级的容器实现，除了拥有BeanFactory的所有支持，而且还提供了诸如统一资源加载，国际化支持，时间发布等特性，默认在容器启动时就会全部加载完成

![image](http://tvax2.sinaimg.cn/large/006A5NwJgy1gk0ir6unnej31260c6wgk.jpg)

## BeanFactory

**BeanFactory**的对象注册与依赖绑定方式

![image](http://tvax1.sinaimg.cn/large/006A5NwJgy1gk0istqgozj30xk0bq411.jpg)

如图所示，BeanFacotry只是一个接口，我们最终肯定需要一个接口的实现来进行实际的Bean管理，而DefaultistableBeanFactory就是这样一个通用的实现类，除了实现BeanFactory接口，还实现了BeanDefinitionRegistry接口，这个接口主要担任Bean注册管理的角色，而BeanFactory只定义如何访问容器内管理的Bean的方法。

#### Bean的Scope

