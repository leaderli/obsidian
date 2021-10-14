一个用于生成java代码的工具类

[官网](https://github.com/square/javapoet)
[JavaDoc](https://square.github.io/javapoet/1.x/javapoet/)

### 使用

```xml
<dependency>
  <groupId>com.squareup</groupId>
  <artifactId>javapoet</artifactId>
  <version>1.13.0</version>
</dependency>

```

### 快速入门

```java
MethodSpec main = MethodSpec.methodBuilder("main")
    .addModifiers(Modifier.PUBLIC, Modifier.STATIC)
    .returns(void.class)
    .addParameter(String[].class, "args")
    .addStatement("$T.out.println($S)", System.class, "Hello, JavaPoet!")
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
    .addModifiers(Modifier.PUBLIC, Modifier.FINAL)
    .addMethod(main)
    .build();

JavaFile javaFile = JavaFile.builder("com.example.helloworld", helloWorld)
    .build();

javaFile.writeTo(System.out);

```

上述代码执行后会生成如下内容
```java
package com.example.helloworld;

import java.lang.String;
import java.lang.System;

public final class HelloWorld {
  public static void main(String[] args) {
    System.out.println("Hello, JavaPoet!");
  }
}
```


可以将源码生成到指定根目录下（作为classpath），会自动生成到对应的包目录下

```java
// path 某个目录下
javaFile.writeTo(new File(path));
```


### for循环

```java
MethodSpec main = MethodSpec.methodBuilder("main")
    .addStatement("int total = 0")
    .beginControlFlow("for (int i = 0; i < 10; i++)")
    .addStatement("total += i")
    .endControlFlow()
    .build();
```

生成如下：

```java
void main() {
  int total = 0;
  for (int i = 0; i < 10; i++) {
    total += i;
  }
}
```


### if语句

```java
MethodSpec main = MethodSpec.methodBuilder("main")
    .addStatement("long now = $T.currentTimeMillis()", System.class)
    .beginControlFlow("if ($T.currentTimeMillis() < now)", System.class)
    .addStatement("$T.out.println($S)", System.class, "Time travelling, woo hoo!")
    .nextControlFlow("else if ($T.currentTimeMillis() == now)", System.class)
    .addStatement("$T.out.println($S)", System.class, "Time stood still!")
    .nextControlFlow("else")
    .addStatement("$T.out.println($S)", System.class, "Ok, time still moving forward")
    .endControlFlow()
    .build();
```

```java
void main() {
  long now = System.currentTimeMillis();
  if (System.currentTimeMillis() < now)  {
    System.out.println("Time travelling, woo hoo!");
  } else if (System.currentTimeMillis() == now) {
    System.out.println("Time stood still!");
  } else {
    System.out.println("Ok, time still moving forward");
  }
}
```

### 占位符

#### `$L`
将`$L`直接替换为文字，不做任何特殊处理

```java
private MethodSpec computeRange(String name, int from, int to, String op) {
  return MethodSpec.methodBuilder(name)
      .returns(int.class)
      .addStatement("int result = 0")
      .beginControlFlow("for (int i = $L; i < $L; i++)", from, to)
      .addStatement("result = result $L i", op)
      .endControlFlow()
      .addStatement("return result")
      .build();
}
```

```java
 int hello() {
    int result = 0;
    for (int i = 1; i < 100; i++) {
      result = result + i;
    }
    return result;
  }
```


#### `$S`

将`$S`替换为字符串，会自动处理转移字符

```java
MethodSpec.methodBuilder("test")
	.returns(void.class)
	.addStatement("String a = $S", new Object[]{null})
	.addStatement("String b = $S", "hello")
	.build()
```

```java
void test() {
    String a = null;
    String b = "hello";
}
```

#### `$T`

将`$T`替换为实际的java类，会自动import

```java
MethodSpec today = MethodSpec.methodBuilder("today")  
        .returns(Date.class)  
        .addStatement("return new $T()", Date.class)  
        .build();  
  
TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")  
        .addModifiers(Modifier.PUBLIC, Modifier.FINAL)  
        .addMethod(today)  
        .build();  
  
JavaFile javaFile = JavaFile.builder("com.example.helloworld", helloWorld)  
        .build();  
  
javaFile.writeTo(System.out);
```

```java
package com.example.helloworld;

import java.util.Date;

public final class HelloWorld {
  Date today() {
    return new Date();
  }
}

```

可是使用暂时不存在的类

```java
ClassName fake = ClassName.get("com.fake", "Fake");  
MethodSpec today = MethodSpec.methodBuilder("today")  
        .returns(fake)  
        .addStatement("return new $T()", fake)  
        .build();
```

```java
import com.fake.Fake;

Fake today() {
    return new Fake();
  }
```

支持泛型

```java
ClassName hoverboard = ClassName.get("com.mattel", "Hoverboard");  
ClassName list = ClassName.get("java.util", "List");  
ClassName arrayList = ClassName.get("java.util", "ArrayList");  
TypeName listOfHoverboards = ParameterizedTypeName.get(list, hoverboard);  
  
MethodSpec beyond = MethodSpec.methodBuilder("beyond")  
        .returns(listOfHoverboards)  
        .addStatement("$T result = new $T<>()", listOfHoverboards, arrayList)  
        .addStatement("result.add(new $T())", hoverboard)  
        .addStatement("result.add(new $T())", hoverboard)  
        .addStatement("result.add(new $T())", hoverboard)  
        .addStatement("return result")  
        .build();  
  
TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")  
        .addModifiers(Modifier.PUBLIC, Modifier.FINAL)  
        .addMethod(beyond)  
        .build();  
  
JavaFile javaFile = JavaFile.builder("com.example.helloworld", helloWorld)  
        .build();  
  
javaFile.writeTo(System.out);
```

```java
package com.example.helloworld;

import com.mattel.Hoverboard;
import java.util.ArrayList;
import java.util.List;

public final class HelloWorld {
  List<Hoverboard> beyond() {
    List<Hoverboard> result = new ArrayList<>();
    result.add(new Hoverboard());
    result.add(new Hoverboard());
    result.add(new Hoverboard());
    return result;
  }
}

```


可以配置为静态引入`static import`

```java
ClassName hoverboard = ClassName.get("com.mattel", "Hoverboard");  
ClassName list = ClassName.get("java.util", "List");  
ClassName arrayList = ClassName.get("java.util", "ArrayList");  
TypeName listOfHoverboards = ParameterizedTypeName.get(list, hoverboard);  
ClassName namedBoards = ClassName.get("com.mattel", "Hoverboard", "Boards");  
  
MethodSpec beyond = MethodSpec.methodBuilder("beyond")  
        .returns(listOfHoverboards)  
        .addStatement("$T result = new $T<>()", listOfHoverboards, arrayList)  
        .addStatement("result.add($T.createNimbus(2000))", hoverboard)  
        .addStatement("result.add($T.createNimbus(\"2001\"))", hoverboard)  
        .addStatement("result.add($T.createNimbus($T.THUNDERBOLT))", hoverboard, namedBoards)  
        .addStatement("$T.sort(result)", Collections.class)  
        .addStatement("return result.isEmpty() ? $T.emptyList() : result", Collections.class)  
        .build();  
  
TypeSpec hello = TypeSpec.classBuilder("HelloWorld")  
        .addMethod(beyond)  
        .build();  
  
JavaFile javaFile = JavaFile.builder("com.example.helloworld", hello)  
        .addStaticImport(hoverboard, "createNimbus")  
        .addStaticImport(namedBoards, "*")  
        .addStaticImport(Collections.class, "*")  
        .build();  
javaFile.writeTo(System.out);
```

```java
package com.example.helloworld;

import static com.mattel.Hoverboard.Boards.*;
import static com.mattel.Hoverboard.createNimbus;
import static java.util.Collections.*;

import com.mattel.Hoverboard;
import java.util.ArrayList;
import java.util.List;

class HelloWorld {
  List<Hoverboard> beyond() {
    List<Hoverboard> result = new ArrayList<>();
    result.add(createNimbus(2000));
    result.add(createNimbus("2001"));
    result.add(createNimbus(THUNDERBOLT));
    sort(result);
    return result.isEmpty() ? emptyList() : result;
  }
}
```


#### `$N`
`$N`可引入其他声明的名称
```java
MethodSpec hexDigit = MethodSpec.methodBuilder("hexDigit")
    .addParameter(int.class, "i")
    .returns(char.class)
    .addStatement("return (char) (i < 10 ? i + '0' : i - 10 + 'a')")
    .build();

MethodSpec byteToHex = MethodSpec.methodBuilder("byteToHex")
    .addParameter(int.class, "b")
    .returns(String.class)
    .addStatement("char[] result = new char[2]")
    .addStatement("result[0] = $N((b >>> 4) & 0xf)", hexDigit)
    .addStatement("result[1] = $N(b & 0xf)", hexDigit)
    .addStatement("return new String(result)")
    .build();

```

### 代码块中的占位符

> I ate 3 tacos

相对位置的占位符
```java
CodeBlock.builder().add("I ate $L $L", 3, "tacos")
```
带位置参数的占位符
```java
CodeBlock.builder().add("I ate $2L $1L", "tacos", 3)
```
使用命令参数的占位符

```java
Map<String, Object> map = new LinkedHashMap<>();
map.put("food", "tacos");
map.put("count", 3);
CodeBlock.builder().addNamed("I ate $count:L $food:L", map)

```

### 抽象方法

```java
MethodSpec flux = MethodSpec.methodBuilder("flux")
    .addModifiers(Modifier.ABSTRACT, Modifier.PROTECTED)
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
    .addModifiers(Modifier.PUBLIC, Modifier.ABSTRACT)
    .addMethod(flux)
    .build();
```

```java
public abstract class HelloWorld {
  protected abstract void flux();
}
```

### 构造器

```java
MethodSpec flux = MethodSpec.constructorBuilder()
    .addModifiers(Modifier.PUBLIC)
    .addParameter(String.class, "greeting")
    .addStatement("this.$N = $N", "greeting", "greeting")
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
    .addModifiers(Modifier.PUBLIC)
    .addField(String.class, "greeting", Modifier.PRIVATE, Modifier.FINAL)
    .addMethod(flux)
    .build();
```

```java
public class HelloWorld {
  private final String greeting;

  public HelloWorld(String greeting) {
    this.greeting = greeting;
  }
}
```

### 方法参数

可以通过两种方式

```java
ParameterSpec android = ParameterSpec.builder(String.class, "android")
    .addModifiers(Modifier.FINAL)
    .build();

MethodSpec welcomeOverlords = MethodSpec.methodBuilder("welcomeOverlords")
    .addParameter(android)
    .addParameter(String.class, "robot", Modifier.FINAL)
    .build();
```

```java
void welcomeOverlords(final String android, final String robot) {
}
```

###  成员变量

也有两种方式创建

```java
FieldSpec android = FieldSpec.builder(String.class, "android")
    .addModifiers(Modifier.PRIVATE, Modifier.FINAL)
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
    .addModifiers(Modifier.PUBLIC)
    .addField(android)
    .addField(String.class, "robot", Modifier.PRIVATE, Modifier.FINAL)
    .build();
```

```
public class HelloWorld {
  private final String android;

  private final String robot;
}
```

成员变量初始化赋值，可和[[#代码块中的占位符]]一样使用

```java
FieldSpec android = FieldSpec.builder(String.class, "android")
    .addModifiers(Modifier.PRIVATE, Modifier.FINAL)
    .initializer("$S + $L", "Lollipop v.", 5.0d)
    .build();
```

```java
private final String android = "Lollipop v." + 5.0;
```

### 接口

接口中的方法必须恒为`PUBLIC ABSTRACT`
接口中的成员变量恒为`PUBLIC STATIC FINAL`


```java
TypeSpec helloWorld = TypeSpec.interfaceBuilder("HelloWorld")
    .addModifiers(Modifier.PUBLIC)
    .addField(FieldSpec.builder(String.class, "ONLY_THING_THAT_IS_CONSTANT")
        .addModifiers(Modifier.PUBLIC, Modifier.STATIC, Modifier.FINAL)
        .initializer("$S", "change")
        .build())
    .addMethod(MethodSpec.methodBuilder("beep")
        .addModifiers(Modifier.PUBLIC, Modifier.ABSTRACT)
        .build())
    .build();
```

```java
public interface HelloWorld {
  String ONLY_THING_THAT_IS_CONSTANT = "change";

  void beep();
}
```

### 枚举

```java
TypeSpec helloWorld = TypeSpec.enumBuilder("Roshambo")
    .addModifiers(Modifier.PUBLIC)
    .addEnumConstant("ROCK")
    .addEnumConstant("SCISSORS")
    .addEnumConstant("PAPER")
    .build();
```

```java
public enum Roshambo {
  ROCK,
  SCISSORS,
  PAPER
}
```

### 匿名内部类


```java
TypeSpec comparator = TypeSpec.anonymousClassBuilder("")
    .addSuperinterface(ParameterizedTypeName.get(Comparator.class, String.class))
    .addMethod(MethodSpec.methodBuilder("compare")
        .addAnnotation(Override.class)
        .addModifiers(Modifier.PUBLIC)
        .addParameter(String.class, "a")
        .addParameter(String.class, "b")
        .returns(int.class)
        .addStatement("return $N.length() - $N.length()", "a", "b")
        .build())
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
    .addMethod(MethodSpec.methodBuilder("sortByLength")
        .addParameter(ParameterizedTypeName.get(List.class, String.class), "strings")
        .addStatement("$T.sort($N, $L)", Collections.class, "strings", comparator)
        .build())
    .build();

```

```java
void sortByLength(List<String> strings) {
  Collections.sort(strings, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
      return a.length() - b.length();
    }
  });
}

```

### 注解

```java
MethodSpec toString = MethodSpec.methodBuilder("toString")
    .addAnnotation(Override.class)
    .returns(String.class)
    .addModifiers(Modifier.PUBLIC)
    .addStatement("return $S", "Hoverboard")
    .build();
```

```java
  @Override
  public String toString() {
    return "Hoverboard";
  }
```

带参数的注解

```java
MethodSpec logRecord = MethodSpec.methodBuilder("recordEvent")
    .addModifiers(Modifier.PUBLIC, Modifier.ABSTRACT)
    .addAnnotation(AnnotationSpec.builder(Headers.class)
        .addMember("accept", "$S", "application/json; charset=utf-8")
        .addMember("userAgent", "$S", "Square Cash")
        .build())
    .addParameter(LogRecord.class, "logRecord")
    .returns(LogReceipt.class)
    .build();
```

```java
@Headers(
    accept = "application/json; charset=utf-8",
    userAgent = "Square Cash"
)
LogReceipt recordEvent(LogRecord logRecord);
```


多次调用`addMember`方法，注解值则会变成list

### 可添加注释

```java
MethodSpec dismiss = MethodSpec.methodBuilder("dismiss")
    .addJavadoc("Hides {@code message} from the caller's history. Other\n"
        + "participants in the conversation will continue to see the\n"
        + "message in their own history unless they also delete it.\n")
    .addJavadoc("\n")
    .addJavadoc("<p>Use {@link #delete($T)} to delete the entire\n"
        + "conversation for all participants.\n", Conversation.class)
    .addModifiers(Modifier.PUBLIC, Modifier.ABSTRACT)
    .addParameter(Message.class, "message")
    .build();
```

```java
  /**
   * Hides {@code message} from the caller's history. Other
   * participants in the conversation will continue to see the
   * message in their own history unless they also delete it.
   *
   * <p>Use {@link #delete(Conversation)} to delete the entire
   * conversation for all participants.
   */
  void dismiss(Message message);

```


### 继承
```java
TypeSpec typeSpec = TypeSpec.classBuilder("Dummy")
  .addSuperinterface(Serializable.class) 
  .superclass(Exception.class) 
  .build();

```