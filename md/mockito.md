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

## 实现原理简析

mockito使用动态代理，代理原始类的方法，使用`when`时不关心方法的调用过程，仅记录方法的签名，然后在`thenReturn`中根据签名缓存执行结果，当实际调用方法时从缓存中取出模拟执行结果。

比如我们尝试去实现一个最简单的上述示例

```java
private static class Invocation {  
    private final Object mock;  
    private final Method method;  
    private final Object[] arguments;  
    private final MethodProxy proxy;  
  
    public Invocation(Object mock, Method method, Object[] args, MethodProxy proxy) {  
        this.mock = mock;  
        this.method = method;  
        this.arguments = copyArgs(args);  
        this.proxy = proxy;  
    }  
  
    private Object[] copyArgs(Object[] args) {  
        Object[] newArgs = new Object[args.length];  
        System.arraycopy(args, 0, newArgs, 0, args.length);  
        return newArgs;  
    }  
  
    @Override  
 public boolean equals(Object obj) {  
        if (obj == null || !obj.getClass().equals(this.getClass())) {  
            return false;  
        }  
        Invocation other = (Invocation) obj;  
        return this.mock == other.mock && this.method.equals(other.method) && this.proxy.equals((other).proxy)  
                && Arrays.deepEquals(arguments, other.arguments);  
    }  
  
    @Override  
 public int hashCode() {  
        return 1;  
    }  
}
```

```java
private static class MockInterceptor implements MethodInterceptor {  
  
  
    @Override  
 public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {  
        Invocation invocation = new Invocation(obj, method, args, proxy);  
        lastInvocation = invocation;  
        return results.get(invocation);  
    }  
}  
  
```

```java
private static class When<T> {  
    public void thenReturn(T retObj) {  
        results.put(lastInvocation, retObj);  
    }  
}
```


```java
public static class Mock {  
    private static final Map<Invocation, Object> results = new HashMap<Invocation, Object>();  
    private static Invocation lastInvocation;  
  
    @SuppressWarnings("unchecked")  
    public static <T> T mock(Class<T> clazz) {  
  
        Enhancer enhancer = new Enhancer();  
        enhancer.setSuperclass(clazz);  
        enhancer.setCallback(new MockInterceptor());  
        return (T) enhancer.create();  
    }  
  
    public static <T> When<T> when(T o) {  
        return new When<T>();  
    }
}
```


测试
```java
List mock = Mock.mock(List.class);  
System.out.println(mock);  
Mock.when(mock.toString()).thenReturn("fuck");  
System.out.println(mock);

//null
//fuck
```

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


