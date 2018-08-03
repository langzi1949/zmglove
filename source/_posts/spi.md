---
title: Java-SPI机制和源码分析
date: 2018-08-02 11:01:00
tags:
    - java
---
SPI（Service Provider Interface），就是为了使不同厂商针对同一个接口做出不同实现，比如最常见的就是`JDBC`中 **mysql**和 **postgresql**，他们都提供了JDBC的实现方式，而`SPI`机制可以为某个接口寻找到具体的实现。
为了使得服务的可插拔，在接口实现时，不能采取硬编码的方式。所以java的`SPI`就派上了用场。<br/>
当服务的提供者提供了一种接口的实现方式，需要在classpath下的`META-INFO/services/`目录里面创建一个以服务接口命名的文件（是接口的全路径名称），这个文件里面的内容就是接口的具体的实现类。当其他的程序需要这个服务的时候，就可以通过查找这个jar包（一般都是以jar包做依赖）`META-INF/services/`中的配置文件，配置文件中有接口的具体实现类名，可以根据这个类名进行加载实例化，就可以使用该服务了。<br/>
JDK中查找服务的工具类是`java.util.ServiceLoader`

### 以JDBC作为例子
在jdbc4.0之前，我们创建数据库连接的时候，通常需要`Class.forName("com.mysql.jdbc.Driver")`这句话进行驱动的加载，然后再获取连接。但是在jdbc4.0以后，就不需要这句话来加载驱动了，而是直接获取连接就行了，这个就是通过`SPI`机制实现的。
<!--more-->
#### JDBC接口定义
Java中已经定义了接口`java.sql.Driver`，并没有提供具体的实现，实现是让各个厂商自己提供的。

#### Mysql实现
在`mysql-connector-java-8.0.9-rc`版本中，通过jar包可以发现在`META-INF/services/`目录中，有个文件`java.sql.Driver`，里面的内容为`com.mysql.cj.jdbc.Driver`，这个就是对jdbc的具体实现类。

#### PostgreSQL实现
在`postgresql-9.1-901-1.jdbc4`版本中，也可以找到相同的配置文件，只是里面的内容为`org.postgresql.Driver`，这个同样也是对jdbc的具体实现类。

#### 使用方法
由于我们已经使用了`SPI`机制，就不需要手动加载驱动了，而是直接通过如下代码:
```java
String url = "jdbc:xxxx://xxxx:port/db";
Connection conn = DriverManager.getConnection(url, userName, password);
// More
```

#### 源码实现
以上的代码并涉及到`SPI`的代码，也没有硬编码进行驱动的加载`Class.forName("com.mysql.jdbc.Driver")`!那上面的代码是如何进行驱动的加载的呢？
关于驱动的加载都在类`DriverManager`中，这个是Java中的实现，用来获取数据库的连接的，在`DriverManager`中有个静态代码块：
```java
static {
        loadInitialDrivers();
        println("JDBC DriverManager initialized");
    }
```
可以查看`loadInitialDrivers`方法
```java
private static void loadInitialDrivers() {
        // 略....

        AccessController.doPrivileged(new PrivilegedAction<Void>() {
            public Void run() {
                // 通过SPI进行加载接口的实现
                ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
                Iterator<Driver> driversIterator = loadedDrivers.iterator();
                try{
                    while(driversIterator.hasNext()) {
                        driversIterator.next();
                    }
                } catch(Throwable t) {
                // Do nothing
                }
                return null;
            }
        });
        // 略.....
    }

```
以上代码的主要步骤：

* 从系统变量中获取有关驱动的定义
* 使用SPI进行驱动的加载
* 遍历使用SPI获取到的具体实现，实例化各个实现类
* 根据第一步获取到的驱动列表来实例化具体实现类

在第2，3步中，通过SPI进行加载，在进行迭代器遍历的时候，`driversIterator.hasNext()`，会搜索classpath下以及jar包中所有的`META-INF/services`目录下的`java.sql.Driver`文件，并找到文件中的实现类的名字，此时并没有实例化具体的实现类。
然后通过`driversIterator.next()`方法会根据驱动名称，完成实例化操作。

#### Debug跟踪
我在测试的代码中跟踪了Driver的驱动加载，如下图所示：
!["SPI-Debug"](/images/spi.jpg)
发现mysql和postgresql的都被加载了。


### SPI的使用步骤总结
通过上面的JDBC的例子，就会发现SPI的使用步骤：
* 接口定义
* 具体的厂商或者开发者提供实现
* 程序员使用

#### 接口定义
例如：`java.sql.Driver`

#### 具体的厂商或者开发者提供实现
在`META-INFO/services/`目录下创建一个接口全路径名称的文件，里面写上接口具体的实现类

#### 程序员使用
```java
ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
//获取迭代器
Iterator<Driver> driversIterator = loadedDrivers.iterator();
//遍历
while(driversIterator.hasNext()) {
    driversIterator.next();
    //可以做具体的业务逻辑
}
```

### SPI相关代码解析
```
//ServiceLoader实现了Iterable接口，可以遍历所有的服务实现者
public final class ServiceLoader<S>
    implements Iterable<S>
{

    //查找配置文件的目录
    private static final String PREFIX = "META-INF/services/";

    //表示要被加载的服务的类或接口
    private final Class<S> service;

    //这个ClassLoader用来定位，加载，实例化服务提供者
    private final ClassLoader loader;

    // 访问控制上下文
    private final AccessControlContext acc;

    // 缓存已经被实例化的服务提供者，按照实例化的顺序存储
    private LinkedHashMap<String,S> providers = new LinkedHashMap<>();

    // 迭代器
    private LazyIterator lookupIterator;

    // ........

    //解析服务提供者配置文件中的一行
    //首先去掉注释校验，然后保存
    //返回下一行行号
    //重复的配置项和已经被实例化的配置项不会被保存
    private int parseLine(Class<?> service, URL u, BufferedReader r, int lc,
                          List<String> names)
        throws IOException, ServiceConfigurationError
    {
        //读取一行
        String ln = r.readLine();
        if (ln == null) {
            return -1;
        }
        //#号代表注释行
        int ci = ln.indexOf('#');
        if (ci >= 0) ln = ln.substring(0, ci);

        //.......

        return lc + 1;
    }

    //解析配置文件，解析指定的url配置文件
    //使用parseLine方法进行解析，未被实例化的服务提供者会被保存到缓存中去
    private Iterator<String> parse(Class<?> service, URL u)
        throws ServiceConfigurationError
    {
        InputStream in = null;
        BufferedReader r = null;
        ArrayList<String> names = new ArrayList<>();
        try {
            in = u.openStream();
            r = new BufferedReader(new InputStreamReader(in, "utf-8"));
            int lc = 1;
            while ((lc = parseLine(service, u, r, lc, names)) >= 0);
        }
        return names.iterator();
    }

    //Private inner class implementing fully-lazy provider lookup
    private class LazyIterator
        implements Iterator<S>
    {
        Class<S> service;//服务提供者接口
        ClassLoader loader;//类加载器
        Enumeration<URL> configs = null;//保存实现类的url
        Iterator<String> pending = null;//保存实现类的全名
        String nextName = null;//迭代器中下一个实现类的全名

        //.......

    }

    //获取迭代器
    //返回遍历服务提供者的迭代器
    //以懒加载的方式加载可用的服务提供者
    //懒加载的实现是：解析配置文件和实例化服务提供者的工作由迭代器本身完成
    public Iterator<S> iterator() {
        return new Iterator<S>() {
            //按照实例化顺序返回已经缓存的服务提供者实例
            Iterator<Map.Entry<String,S>> knownProviders
                = providers.entrySet().iterator();

            public boolean hasNext() {
                if (knownProviders.hasNext())
                    return true;
                return lookupIterator.hasNext();
            }

            public S next() {
                if (knownProviders.hasNext())
                    return knownProviders.next().getValue();
                return lookupIterator.next();
            }

            public void remove() {
                throw new UnsupportedOperationException();
            }

        };
    }

```

`ServiceLoader`不是实例化以后，就去读取配置文件中的具体实现，并进行实例化；而是等到使用迭代器去遍历的时候，才会加载对应的配置文件去解析，调用`hasNext`方法的时候会去加载配置文件进行解析，调用`next`方法的时候进行实例化并缓存。

所有的配置文件只会加载一次，服务提供者也只会被实例化一次，重新加载配置文件可使用reload方法。

### SPI缺点
通过上面的解析，可以发现，我们使用SPI查找具体的实现的时候，需要遍历所有的实现，并实例化，然后我们在循环中才能找到我们需要实现。这应该也是最大的缺点，需要把所有的实现都实例化了，即便我们不需要，也都给实例化了。
