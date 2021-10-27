#java #junit #mock


一个用来模拟测试的框架，[官方文档](https://javadoc.io/static/org.mockito/mockito-core/4.0.0/org/mockito/Mockito.html)

## 快速入门
```xml
<dependency>
  <groupId>org.mockito</groupId>
  <artifactId>mockito-core</artifactId>
  <version>1.10.19</version>
</dependency>
```

后续代码统一 `import static`
```java
import static org.mockito.Mockito.*;


List mockedList = mock(List.class);  
when(mockedList.get(0)).thenReturn("hello");  
assertEquals("hello", mockedList.get(0));
```

## 实现原理

mockito使用动态代理，


## 常用API


```java
import static org.mockito.Mockito.*;

//mock creation
List mockedList = mock(List.class);

//using mock object
mockedList.add("one");
mockedList.clear();

//断言前面使用了add方法，且参数为one
verify(mockedList).add("one");
//断言前面使用了clear方法
verify(mockedList).clear();

```


