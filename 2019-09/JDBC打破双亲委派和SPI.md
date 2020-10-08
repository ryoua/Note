### JDBC打破双亲委派和SPI 

当我们使用DriverManager.connection()方法去连接数据库的时候, 我们并没有去指定连接哪一个数据库, 其实DriverManager是有中有一个list保存了所有的引入的Driver, 然后他会做一个轮询, 看哪一个可以连上.如果连上就直接使用并返回, 如果都不行, 就返回错误.

   但是这里就有一个问题, DriverManager是如何知道你引入了哪些Driver的, 答案就是每当我们引入一个Driver, 其jar包目录下就有一个META-INF文件夹, 下面有一个java.sql.Driver文件, 里面保存了对应的Drvier类路径, 比如H2数据库, 其内容就是org.h2.Driver, 这就是SPI的配置, 当我们配置好之后, DriverManager会通过ServiceLoader去加载对应的Driver类, 每个数据库的Driver类里面会有一个static代码块执行load方法, 去把自己注册到DriverManager的list中

   有的时候我们会用Class.forname, 这个主要的目的是有的jdbc版本太老, 不支持spi, 所以用Class.forname来手动注册

   当我们加载h2和mysql驱动的时候, 由于DriverManager是jdk自带的类, 所以会使用bootstrap加载器去加载, 而由于全盘委托, 对应数据库驱动的类也会使用当前类加载器去加载, 但是h2和mysql等数据库驱动类是用户类, 用bootstrap加载器去加载是不合适的, 这个时候就要替换加载器, 使用AppClassloader来替换当前线程类加载器.