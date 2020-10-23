在学习Spring AOP之前，我们需要先了解静态代理和动态代理

#### 静态代理

静态代理，即代理类和被代理的类实现了同样的接口，代理类同时持有被代理类的引用，这样，当我们需要调用目标方法时，可以通过调用代理类的目标方法来实现

举个例子，首先我先定义一个接口

```java
public interface Buy {
  void buyHouse();
}
```

然后定义一个实现类：

```java
public class BuyHouse implement Buy {
  void buyHouse() {
   System.out.println("买房");
  }
}
```

当我们不使用静态代理时，我们可以直接通过BuyHouse的buyHouse方法来买房，但是当我们在买房时需要一大堆手续，但是我们并不想做，所以我们需要一个中介，也就是代理类，来帮我们完成这些事情，代理类实现如下

```java
public class BuyHouseProxy implement Buy {
  private Buy buy;

  public BuyHouseProxy(Buy buy) {
    this.buy = buy;
  }
   
  void buyHouse() {
    System.out.println("完成一大堆手续");
    buy.buyHouse();
  }
}
```

接下来就是使用的方法

```java
public class Application {
  public static void main(String[] args) {
    Buy buy = new BuyHouse();
    BuyHouseProxy proxy = new BuyHouseProxy(buy);
    proxy.buyHouse();
  }
}
```

这样，我们就完成了静态代理的实现。可以看出实现还是比较简单的，从AOP的角度来看，静态代理有以下几个硬伤

1. 代理的类必须实现了接口
2. 每有一个需要用到AOP的类，就必须为之创建一个代理类，后期会很麻烦

#### 动态代理

动态代理就是为了解决静态代理上述的两个问题而生的，动态代理根据实现方式的不同可以分为JDK动态代理和cglib动态代理

- jdk动态代理：利用反射机制生成一个实现代理接口的类，在调用具体方法之前调用InvokeHandler来处理
- cglib动态代理：利用ASM开源包来将需要代理的类的class文件加载进来，通过修改其字节码生成子类的实现代理

从概念上可以看出，JDK代理只能实现接口的类生成代理，cglib可以对类实现代理，无需接口，但是由于采用的是子类继承的方法，所以不能代理final修饰的类

jdk动态代理的实现如下:

```java
// 接口
public interface IUserDao {
 void save();
 void find();
}

//目标对象
 class UserDao implements IUserDao{
 @Override
 public void save() {
 		System.out.println("模拟： 保存用户！");
 }

 @Override
 public void find() {
 		System.out.println("查询");
 }
}

/**
 * 动态代理：
 * 代理工厂，给多个目标对象生成代理对象！
 */
class ProxyFactory {
 // 接收一个目标对象
 private Object target;

 public ProxyFactory(Object target) {
 		this.target = target;
 }

 // 返回对目标对象(target)代理后的对象(proxy)
 public Object getProxyInstance() {
 Object proxy = Proxy.newProxyInstance(
  target.getClass().getClassLoader(), // 目标对象使用的类加载器
  target.getClass().getInterfaces(),  // 目标对象实现的所有接口
  new InvocationHandler() {  // 执行代理对象方法时候触发

  @Override
  public Object invoke(Object proxy, Method method, Object[] args)
   throws Throwable {
   // 获取当前执行的方法的方法名
   String methodName = method.getName();
   // 方法返回值
   Object result = null;
   if ("find".equals(methodName)) {
   // 直接调用目标对象方法
   result = method.invoke(target, args);
   } else {
   System.out.println("开启事务...");
   // 执行目标对象方法
   result = method.invoke(target, args);
   System.out.println("提交事务...");
   }
   return result;
  }
  }
 );
 return proxy;
 }
}
```

在运行测试类中创建测试类对象代码中

```java
IUserDao proxy = (IUserDao)new ProxyFactory(target).getProxyInstance();
```

其实是jdk动态生成了一个类去实现接口,隐藏了这个过程:

```java
class $jdkProxy implements IUserDao{}
```

使用**jdk**生成的动态代理的前提是目标类必须有实现的接口。但这里又引入一个问题,如果某个类没有实现接口,就不能使用jdk动态代理,所以Cglib代理就是解决这个问题的。

Cglib是以动态生成的子类继承目标的方式实现，在运行期动态的在内存中构建一个子类，如下:

```java
public class UserDao{}

//Cglib是以动态生成的子类继承目标的方式实现,程序执行时,隐藏了下面的过程
public class $Cglib_Proxy_class extends UserDao{}
```

在了解了静态代理和动态代理的概念后，就可以来学习Spring AOP了

## Spring AOP

AOP总共存在三种实现方式，分别是静态AOP，动态AOP，动态字节码生成

- 静态AOP：在编译时，切面直接以字节码的形式编译到目标字节码文件中，对性能无影响，但是灵活性较低
- 动态AOP：在运行时，目标类加载后，为接口动态生成代理类，将切面织入到代理类中，比静态AOP灵活，但是需要实现接口，同时对性能也有一定影响
- 动态字节码生成：cglib实现，在运行时，目标类加载后，动态生成目标类的子类，将切面逻辑加到目标字节码里，即使目标类没有实现接口也可以进行生成，但是无法继承final修饰的方法

在这三种方法中，分别对应了上文中静态代理和动态代理的对应实现，其实AOP的核心思想就是对目标类进行代理，只不过是代理的方式不一样罢了。

在继续深入之前，需要学习一下AOP的术语

- 通知（Advice）：描述了切面何时执行，以及如何增强处理，比如@Before，@After
- 连接点（join point）：表示切入的方法，表示切入的一个点，可以是方法调用，也可以是异常抛出，比如hello方法的方法调用就是一个连接点。
- 切点（Point cut）：表示连接点的集合，在Spring Aop中我们通常通过通配符来配置切点，比如 ”com.test.*“
- 切面（Aspect）: 上面的结合
- 织入（Weaving）：就是将增强的功能写入到目标类中。

接下来具一个具体的使用案例，其他的就不展开说了

```java
@Aspect //该注解标示该类为切面类 
@Component //注入依赖
public class ConfigDeliveryAspect {
  private static Logger logger = LoggerFactory.getLogger(ConfigDeliveryAspect.class);

  @Resource
  private ConfigDownDao configDownDao;

  //标注该方法体为后置通知，当执行该方法体目标方法执行成功后
  @AfterReturning(returning = "retVal", pointcut = "within(zte.cdn.oms..*) && @annotation(ti)")
  public void deliver(JoinPoint jp, Object retVal, TaskInfo ti) {
    Object[] param = jp.getArgs();//获取目标方法体参数
    List<String> deviceList = new ArrayList<>();
    List<String> deviceTypeList = new ArrayList<>();
    if (setParam(jp, retVal, param, deviceList, deviceTypeList)) return;

    for (int i = 0; i < deviceList.size(); i++) {
      String devId = deviceList.get(i);
      String devType = deviceTypeList.get(i);
      Task task = new Task();
      if (StringUtils.equalsIgnoreCase("json", ti.messagetype())) {
        setTask(retVal, ti, operBean, devId, devType, task);
        MessageEventInit.publish(task);
      }
    }
  }
}
```

上面的切面类实现的主要功能是任务下发的切面，对于写有@TaskInfo注解的方法，Spring Aop都会进行增强，同时使用了AfterReturning通知，当处理完参数后，进行aop处理，获取方法执行完的参数，进行拼接后进行http下发。