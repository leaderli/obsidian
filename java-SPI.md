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

## 应用
- JDBC DriverManager
