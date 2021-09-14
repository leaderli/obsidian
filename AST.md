#java #树 #设计模式


## 概述

抽象语法树（Abstract Syntax Tree, AST）是源代码语法结构的一种抽象表示。它以树状的形式表现编程语言的结构，树的每个节点ASTNode都表示源码中的一个结构。AST把java中的各种元素比如类、属性、方法、代码块、注解、注释等等定义成相应的对象，在编译器编译代码的过程中，语法分析器首先通过AST将源码分析成一个语法树。

## 示例

```xml
<dependency>  
   <groupId>com.github.javaparser</groupId>  
   <artifactId>javaparser-core</artifactId>  
   <version>3.22.1</version>  
</dependency>
```


需要分析的源文件

```java
//TestJavaParser.java
public class TestJavaParser {  
  
    private int age = 18;  
  
    public int m1(int age) {  
  
        if (age > 18) {  
            return age;  
        }  
  
        if (age > 0) {  
            return 5;  
        }  
        return this.age;  
    }  
}
```

一个分析方法`m1`有几个返回值的示例

```java

```


`javaparser`使用访问者模式对语法树进行遍历访问，下面是一些常用的`visitor`

- `ReturnStmt` 含有`return`的语法树，例如`return 1;`
- `IntegerLiteralExpr`   int字面量，例如`int a = 1`中`1`即为`IntegerLiteralExpr`