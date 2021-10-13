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