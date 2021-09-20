#javaslang #vars 

**[VAVR](https://link.zhihu.com/?target=https%3A//github.com/vavr-io/vavr)**，一个扩展函数式编程的第三方库


```xml
<dependency>
    <groupId>io.vavr</groupId>
    <artifactId>vavr</artifactId>
    <version>0.10.3</version>
</dependency>
```

[API文档](https://docs.vavr.io/)
## 集合
Java 的集合库本身是可变的，显然违背了函数式编程的基本特性 - 不可变，为此 VAVR 设计了一套全新的集合库，使用体验无限接近于 Scala。


```java
io.vavr.collection.List.of(1, 2, 3, 4, 5)
    .filter(i -> i > 3)
    .map(i -> i * 2);
```

往集合追加数据会产生新的集合，从而保证不可变

```java
List<Integer> list = List.of(1,2)  
        .appendAll(List.of(3, 4))  
        .appendAll(List.of(5, 6))  
        .append(7);  
  
```

强大的兼容性，可以非常方便的与 Java 标准集合库进行转换

```java
java.util.List<Integer> javaList = java.util.List.of(1, 2, 3);
java.util.List<Integer> javaList2 = io.vavr.collection.List.ofAll(javaList)
    .filter(i -> i > 1)
    .map(i -> i * 2)
    .toJavaList();
```


```java
/**
* 使用了lomock
*/
@Data
class User {
  private Long id;
  private String name;
  private Integer age;
}
```

```java
List.of(new User()).filter(u -> u.getAge() >= 18)  
        .groupBy(User::getAge)  
        .mapValues(usersGroup -> usersGroup.map(User::getName));
```

## 元组

元组类似一个数组，可以存放不同类型的对象并维持其类型信息，这样在取值时就不用 cast 了。

```java
Tuple2<Integer, String> of = Tuple.of(1, "2");  
System.out.println(of._1);  
System.out.println(of._2);
```
目前，VAVR 最多支持构造八元组，也就是支持最多 8 个类型

## 