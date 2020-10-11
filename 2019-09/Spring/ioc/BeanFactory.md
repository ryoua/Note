![image](http://tvax2.sinaimg.cn/large/006A5NwJgy1gjjw0u8kcvj30xy0aaabo.jpg)

> Spring的Ioc容器是一个Ioc Service Provider, 但是这只是他被叫做Ioc容器的部分原因, 更重要的还是容器, Spring在Ioc功能的基础上, 还提供了很多其他功能, 比如对象生命周期管理, 线程管理, 企业服务集成, AOP支持等
>
> Spring提供了两种容器类型: BeanFactory和ApplicationContext
>
> * BeanFactory: 基础的Ioc容器, 提供完整的Ioc支持, 默认采用延迟初始化策略(Lazy-Load), 在资源有限, 且功能要求不是很高的场景, 比较合适
> * ApplicationContext: 在BeanFactory的基础上构建, 除了BeanFactory的所有支持, 还提供了其他高级特性, 比如事件发布, 国际化支持等, 默认采用立即加载, 即在容器启动后, 默认全部初始化并绑定完成, 所以相对BeanFactory来说,  需要的资源更多, 但是提供的功能也更多

# BeanFactory

