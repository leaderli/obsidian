#java 

一个用于方便操作反射的工具类

```xml
<dependency>
  <groupId>org.jooq</groupId>
  <artifactId>joor-java-8</artifactId>
  <version>0.9.13</version>
</dependency>

```


所有示例都统一`static import`

```java
import static org.joor.Reflect.*;

String world = onClass("java.lang.String") // 类似 Class.forName()，也可以直接用onClass(String.class)
                .create("Hello World")     // 调用指定参数的构造器
                .call("substring", 6)      // 调用指定名称和参数类型的方法,改方法支持链式调用
                .get();                    // 返回call方法执行的结果,返回最近的一个非void方法的执行结果

```

```java
public interface StringProxy {
  String substring(int beginIndex);
}

String substring = onClass("java.lang.String")
                    .create("Hello World")
                    .as(StringProxy.class) // 创建一个代理接口，方便调用，代理接口的方法需要和实际反射的方法的签名完全一致
                    .substring(6);         // 通过代理接口调用实际反射的类的方法

```