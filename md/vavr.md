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

//快速生成一个集合
List.range(0,10)
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

## Option
在 VAVR 中，`Option` 是一个 interface，它的具体实现有 Some 和 None
-   Some: 代表有值
-   None: 代表没有值
```java
Object a = null;  
System.out.println(Option.of(a));  
a= 1;  
System.out.println(Option.of(a));

```
>None
>Some(1)



```java
  
import io.vavr.control.Option;  
import org.assertj.core.api.Assertions;  
import org.junit.Test;  
  
public class TestVavr {  
  
  
	 @Test  
	 public void test() {  
		 
		String result = Option.of("hello")
			.map(str -> (String) null)  
			.getOrElse(() -> "world");  
		Assertions.assertThat(result).isNull();  
	}  
}
```

```java
boolean exists = Option.of("ok").exists(str -> str.equals("ok"));
boolean contains = Option.of("ok").contains("ok");
//考虑到与标准库的兼容，Option 可以很方便的与 Optional 进行互转
Option.of("toJava").toJavaOptional();
Option.ofOptional(Optional.empty());
```

## Try

Try 也是个接口， 具体的实现是 Success 或 Failure

-   Success：代表执行没有异常
-   Failure：代表执行出现异常

Try 和 Option 类似，也类似于一个「容器」，只不过它容纳的是**可能出错的行为**，你是不是马上就想到了 try..catch 结构?

```java
Try.of(() -> 1 / 0)
    .andThen(r -> System.out.println("and then " + r))
    .onFailure(error -> System.out.println("failure" + error.getMessage()))
    .andFinally(() -> {
      System.out.println("finally");
    });
```

通过 Try 的 recoverWith 方法，我们可以很优雅的实现降级策略

```java
@Test
public void testTryWithRecover() {
    Assert.assertEquals("NPE", testTryWithRecover(new NullPointerException()));
    Assert.assertEquals("IllegalState", testTryWithRecover(new IllegalStateException()));
    Assert.assertEquals("Unknown", testTryWithRecover(new RuntimeException()));
}

private String testTryWithRecover(Exception e) {
    return (String) Try.of(() -> {
      throw e;
    })
      .recoverWith(NullPointerException.class, Try.of(() -> "NPE"))
      .recoverWith(IllegalStateException.class, Try.of(() -> "IllegalState"))
      .recoverWith(RuntimeException.class, Try.of(() -> "Unknown"))
      .get();
}
```

对于 Try 的计算结果，可以通过 map 进行转换，也可以很方便的与 Option 进行转换。

```java
@Test
public void testTryMap() {
    String res = Try.of(() -> "hello world")
      .map(String::toUpperCase)
      .toOption()
      .getOrElse(() -> "default");
    Assert.assertEquals("HELLO WORLD", res);
}
```

## Future

这个 Future 可不是 `java.util.concurrent.Future`，但它们都是对异步计算结果的一个抽象。

vavr 的 `Future` 提供了比 `java.util.concurrent.Future` 更友好的回调机制

-   onFailure 失败的回调
-   onSuccess 成功的回调

```java
@Test  
public void testFutureFailure() {  
    final String word = "hello world";  
    Future.of(Executors.newFixedThreadPool(1), () -> word)  
		.onFailure(throwable -> Assert.fail("不应该走到 failure 分支"))  
		.onSuccess(result -> Assert.assertEquals(word, result));  
}
```
它也可以和 Java 的 CompleableFuture 互转

```java
Future.of(Executors.newFixedThreadPool(1), () -> "toJava").toCompletableFuture();

Future.fromCompletableFuture(CompletableFuture.runAsync(() -> {}));
```


## Either
Either 它表示某个值可能为两种类型中的一种，比如下面的 `compute()` 函数的 Either 返回值代表结构可能为 Exception 或 String。通常用 right 代表正确的值（英文 right 有正确的意思）

```java
public Either<Exception, String> compute() {  
 return Either.right("123");  
}  
@Test  
public void test() {  
    Either<Exception, String> either = compute();  
  
    // 异常值  
 if (either.isLeft()) {  
        Exception exception = compute().getLeft();  
        throw new RuntimeException(exception);  
    }  
  
    // 正确值  
 if (either.isRight()) {  
        String result = compute().get();  
        System.out.println("result:"+result);  
        // ...  
 }  
}
```

## Lazy

```java
Lazy<Double> lazy = Lazy.of(Math::random);
lazy.isEvaluated(); // = false
lazy.get();         // = 0.123 (random generated)
lazy.isEvaluated(); // = true
lazy.get();         // = 0.123 (memoized)
```

## Validation
同时校验多个属性，而不是在首次校验错误时，就结束

```java
import io.vavr.collection.CharSeq;
import io.vavr.collection.Seq;
import io.vavr.control.Validation;
import org.junit.Test;

public class TestVavr {
    public static class Person {

        public final String name;
        public final int age;

        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }

        @Override
        public String toString() {
            return "Person(" + name + ", " + age + ")";
        }

    }

    public static class PersonValidator {

        private static final String VALID_NAME_CHARS = "[a-zA-Z ]";
        private static final int MIN_AGE = 0;

        public Validation<Seq<String>, Person> validatePerson(String name, int age) {
            return Validation.combine(validateName(name), validateAge(age)).ap(Person::new);
        }

        private Validation<String, String> validateName(String name) {
            return CharSeq.of(name).replaceAll(VALID_NAME_CHARS, "").transform(seq -> seq.isEmpty()
                    ? Validation.valid(name)
                    : Validation.invalid("Name contains invalid characters: '"
                    + seq.distinct().sorted() + "'"));
        }

        private Validation<String, Integer> validateAge(int age) {
            return age < MIN_AGE
                    ? Validation.invalid("Age must be at least " + MIN_AGE)
                    : Validation.valid(age);
        }

    }

    @Test
    public void test() {
        PersonValidator personValidator = new PersonValidator();

        // Valid(Person(John Doe, 30))
        Validation<Seq<String>, Person> valid = personValidator.validatePerson("John Doe", 30);
        if (valid.isValid()) {
            System.out.println(valid.get());

        }

         // Invalid(List(Name contains invalid characters: '!4?', Age must be greater than 0))
        Validation<Seq<String>, Person> invalid = personValidator.validatePerson("John? Doe!4", -1);
        if (invalid.isInvalid()) {
            System.out.println(invalid.getError());
        }

    }

}
```
## 模式匹配
这里的模式指的是**数据结构的组成模式**

-   Match(...)，Case(...)，$(...) 都是 `io.vavr.API` 的静态方法，用于模拟「模式匹配」的语法  
    
-   最后一个 $() 表示匹配除了上面之外的所有情况
```java

import static io.vavr.API.*;

public String bmiFormat(double height, double weight) {
	double bmi = weight / (height * height);
	return Match(bmi).of(
			// else if (bmi < 18.5)
			Case($(v -> v < 18.5), () -> "有些许晃荡！"),
			// else if (bmi < 25)
			Case($(v -> v < 25), () -> "继续加油哦！"),
			// else if (bmi < 30)
			Case($(v -> v < 30), () -> "你是真的稳！"),
			// else
			Case($(), () -> "难受！")
	);
}
```

