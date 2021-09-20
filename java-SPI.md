## 概述

SPI（Service Provider Interface）,是JDK内置的一种服务提供发现机制，比如说`java.sql.Driver`接口，不同的数据库对同一个接口有不同的实现，而SPI就是为某个接口寻找服务实现。将装配的控制权移到程序之外。
![[Java_SPI机制.png]]

当服务的提供者提供了一种接口的实现之后，需要在classpath下的[[maven#META-INF|META-INF/services/]]目录中创建一个以服务接口命名的文件，这个文件里的内容就是接口的具体的实现类。当其他的程序需要这个服务的时候，就剋有通过查找这个jar包的[[maven#META-INF|META-INF/services/]]中的配置文件，配置文件中有接口的具体实现类名，可以根据这个类名进行加载实例化，就可以使用该服务了。JDK中查找服务的实现的工具类是：`java.util.ServiceLoader`。


 ## SPI机制的简单示例
 
定义一个接口
```java
package com.leaderli.lidemo;  
  
public interface ISpiDemo {  
  
    void print();  
}
```
定义实现类
```java
package com.leaderli.lidemo;  
  
public class SpiDemo implements ISpiDemo{  
    @Override  
 public void print() {  
        System.out.println("fuck");  
    }  
}
```

新建文件`src/main/resources/META-INF/services/com.leaderli.lidemo.ISpiDemo`
```txt
com.leaderli.lidemo.SpiDemo
```
使用[[maven#META-INF|maven]]打包

在测试项目中引入该jar包，然后编写测试类

```java
public static void main(String[] args) {  
    ServiceLoader<ISpiDemo> serviceLoader = ServiceLoader.load(ISpiDemo.class);  
    for (ISpiDemo demo : serviceLoader) {  
        demo.print();  
    }  
}
```

## 源码

前面的过程都是在查找类加载器

```java
public static <S> ServiceLoader<S> load(Class<S> service) {  
    ClassLoader cl = Thread.currentThread().getContextClassLoader();  
    return ServiceLoader.load(service, cl);  
}

public static <S> ServiceLoader<S> load(Class<S> service,  
                                        ClassLoader loader)  
{  
    return new ServiceLoader<>(service, loader);  
}

private ServiceLoader(Class<S> svc, ClassLoader cl) {  
    service = Objects.requireNonNull(svc, "Service interface cannot be null");  
    loader = (cl == null) ? ClassLoader.getSystemClassLoader() : cl;  
    acc = (System.getSecurityManager() != null) ? AccessController.getContext() : null;  
    reload();  
}

public void reload() {  
    providers.clear();  
    lookupIterator = new LazyIterator(service, loader);  
}

```

reload中生成了一个LazyIterator，SPI仅在需要的时候才会去加载，其中主要的代码如下

```java
private static final String PREFIX = "META-INF/services/";

private boolean hasNextService() {  
    if (nextName != null) {  
        return true;  
    }  
    if (configs == null) {  
        try {  
			//根据 META-INF/services/接口的全限定名 这个文件
            String fullName = PREFIX + service.getName();  
            if (loader == null){
                configs = ClassLoader.getSystemResources(fullName);  
			}  
            else  {
 				configs = loader.getResources(fullName);  
			}
        } catch (IOException x) {  
            fail(service, "Error locating configuration files", x);  
        }  
    }  
    while ((pending == null) || !pending.hasNext()) {  
        if (!configs.hasMoreElements()) {  
            return false;  
        }  
		//解析文件内容，找到所有实现类的全限定名，会剔除重复的
        pending = parse(service, configs.nextElement());  
    }  
    nextName = pending.next();  
    return true;  
}  
  
private S nextService() {  
    if (!hasNextService())  
        throw new NoSuchElementException();  
    String cn = nextName;  
    nextName = null;  
    Class<?> c = null;  
    try {  
        c = Class.forName(cn, false, loader);  
    } catch (ClassNotFoundException x) {  
        fail(service,  
             "Provider " + cn + " not found");  
    }  
    if (!service.isAssignableFrom(c)) {  
        fail(service,  
             "Provider " + cn + " not a subtype");  
    }  
    try {  
		//使用默认构造器进行实现类的初始化加载
        S p = service.cast(c.newInstance());  
        providers.put(cn, p);  
        return p;  
    } catch (Throwable x) {  
        fail(service,   "Provider " + cn + " could not be instantiated",  x);  
    }  
    throw new Error();          // This cannot happen  
}
```


## 应用
- JDBC DriverManager


## SPI机制的缺陷

通过上面的解析，可以发现，我们使用SPI机制的缺陷：

-   不能按需加载，需要遍历所有的实现，并实例化，然后在循环中才能找到我们需要的实现。如果不想用某些实现类，或者某些类实例化很耗时，它也被载入并实例化了，这就造成了浪费。
    
-   获取某个实现类的方式不够灵活，只能通过 Iterator 形式获取，不能根据某个参数来获取对应的实现类。
    
-   多个并发多线程使用 ServiceLoader 类的实例是不安全的。


我们可以根据其实现原理，来合理化构建一个适合自己使用的SPI服务。
