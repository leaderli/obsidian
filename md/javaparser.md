[官方地址](https://github.com/javaparser/javaparser)
## 示例

```xml
<dependency>  
   <groupId>com.github.javaparser</groupId>  
   <artifactId>javaparser-core</artifactId>  
   <version>3.22.1</version>  
</dependency>
```



### 打印出语法树的结构

#### 使用yaml结构
```java

CompilationUnit cu = StaticJavaParser.parse("class X { int x; }");  
YamlPrinter printer = new YamlPrinter(true);
System.out.println(printer.output(cu));
```

输出结果为

```yaml
---
root(Type=CompilationUnit): 
    types: 
        - type(Type=ClassOrInterfaceDeclaration): 
            isInterface: "false"
            name(Type=SimpleName): 
                identifier: "X"
            members: 
                - member(Type=FieldDeclaration): 
                    variables: 
                        - variable(Type=VariableDeclarator): 
                            name(Type=SimpleName): 
                                identifier: "x"
                            type(Type=PrimitiveType): 
                                type: "INT"
...

```

#### 使用[dot](https://en.wikipedia.org/wiki/DOT_(graph_description_language))图形语法

```java
 CompilationUnit cu = StaticJavaParser.parse("class X { int x; }");  
 DotPrinter printer = new DotPrinter(true);  
 FileWriter fileWriter = new FileWriter("ast.dot");  
 PrintWriter printWriter = new PrintWriter(fileWriter);  
 printWriter.print(printer.output(cu));
```

生成的文件，可通过[在线预览](https://dreampuf.github.io/GraphvizOnline/#digraph)dot文件，生成的图像如下

![[graphviz.svg]]

#### 使用xml

```java
CompilationUnit cu = StaticJavaParser.parse("class X { int x; }");  
XmlPrinter printer = new XmlPrinter(true);  
System.out.println(printer.output(cu));
```

```xml
<root type='CompilationUnit'>  
    <types>  
        <type type='ClassOrInterfaceDeclaration' isInterface='false'>  
            <name type='SimpleName' identifier='X'></name>  
            <members>  
                <member type='FieldDeclaration'>  
                    <variables>  
                        <variable type='VariableDeclarator'>  
                            <name type='SimpleName' identifier='x'></name>  
                            <type type='PrimitiveType' type='INT'></type>  
                        </variable>  
                    </variables>  
                </member>  
            </members>  
        </type>  
    </types>  
</root>
```

###   一个分析返回值是否正确的示例

去解析一个方法`m1`有几个返回值的从而进行提前校验异常规则的一个示例。可先打印出语法树的结构，然后使用指定类型的`visitor`去访问
需要分析的源文件

```java
//TestJavaParser.java
package com.leaderli.demo;  
  
import java.util.Random;  
  
public class TestJavaParser {  
  
    private int age = 2;  
  
    public int m1() {  
  
        int ran = new Random().nextInt(100);  
  
        int temp = 3;  
        if (ran < 10) {  
            return 1;  
        }  
        if (ran < 20) {  
            return age;  
        }  
        if (ran > 40) {  
            temp = 4;  
        }  
        return temp;  
    }  
}
```

```java
package com.leaderli.demo;  


import com.github.javaparser.JavaParser;
import com.github.javaparser.ast.CompilationUnit;
import com.github.javaparser.ast.Node;
import com.github.javaparser.ast.body.FieldDeclaration;
import com.github.javaparser.ast.body.MethodDeclaration;
import com.github.javaparser.ast.body.VariableDeclarator;
import com.github.javaparser.ast.expr.*;
import com.github.javaparser.ast.stmt.ReturnStmt;
import com.github.javaparser.ast.visitor.VoidVisitorAdapter;
import org.junit.Test;

import java.io.File;
import java.io.FileNotFoundException;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

public class TestJavaParser {
    MethodDeclaration m1;

    @Test
    public void test() throws FileNotFoundException {


        JavaParser javaParser = new JavaParser();
        File file = new File("src/main/java/com/leaderli/demo/TestJavaParser.java");
        Optional<CompilationUnit> result = javaParser.parse(file).getResult();

        if (!result.isPresent()) {
            return;
        }

        CompilationUnit cu = result.get();
        List<Integer> returns = new ArrayList<>();

        new VoidVisitorAdapter<Object>() {
            @Override
            public void visit(MethodDeclaration n, Object arg) {
                if ("m1".equals(n.getName().toString())) {
                    m1 = n;
                }
            }
        }.visit(cu, null);

        new VoidVisitorAdapter<Object>() {

            @Override
            public void visit(ReturnStmt returnStmt, Object arg) {
                //打印return语法节点
                System.out.println("ReturnStmt:" + returnStmt);
                for (Node childNode : returnStmt.getChildNodes()) {
                    if (childNode instanceof NameExpr) {
                        //返回语法树使用了变量(成员变量和局部变量，不包括使用this引用的),则查找变量声明和赋值的地方
                        String local = ((NameExpr) childNode).getName().asString();

                        new VoidVisitorAdapter<Object>() {

                            @Override
                            public void visit(FieldDeclaration fieldDeclaration, Object arg) {
                                System.out.println("[" + returnStmt + "] FieldDeclaration:" + fieldDeclaration);
                                new VoidVisitorAdapter<Object>() {
                                    @Override
                                    public void visit(VariableDeclarator variableDeclarator, Object arg) {
                                        boolean find = false;
                                        for (Node node : variableDeclarator.getChildNodes()) {
                                            if (node instanceof SimpleName) {
                                                //是否为当前需要查找找到指定的变量声明名称
                                                find = (local.equals(((SimpleName) node).asString()));
                                            } else {
                                                if (find && node instanceof IntegerLiteralExpr) {
                                                    //若是则找到变量声明赋值的字面量
                                                    int value = ((IntegerLiteralExpr) node).asNumber().intValue();
                                                    returns.add(value);
                                                }
                                            }

                                        }
                                    }
                                }.visit(fieldDeclaration, null);
                            }

                            @Override
                            public void visit(VariableDeclarationExpr variableDeclarationExpr, Object arg) {

                                //变量声明
                                System.out.println("[" + returnStmt + "] VariableDeclarationExpr:" + variableDeclarationExpr);

                                new VoidVisitorAdapter<Object>() {
                                    @Override
                                    public void visit(VariableDeclarator variableDeclarator, Object arg) {
                                        boolean find = false;
                                        for (Node node : variableDeclarator.getChildNodes()) {
                                            if (node instanceof SimpleName) {
                                                //是否为当前需要查找找到指定的变量声明名称
                                                find = (local.equals(((SimpleName) node).asString()));
                                            } else {
                                                if (find && node instanceof IntegerLiteralExpr) {
                                                    //若是则找到变量声明赋值的字面量
                                                    int value = ((IntegerLiteralExpr) node).asNumber().intValue();
                                                    returns.add(value);
                                                }
                                            }

                                        }
                                    }
                                }.visit(variableDeclarationExpr, null);
                            }

                            @Override
                            public void visit(AssignExpr assignExpr, Object arg) {
                                //变量赋值
                                System.out.println("[" + local + "]" + assignExpr);

                                boolean find = false;
                                for (Node node : assignExpr.getChildNodes()) {

                                    if (node instanceof NameExpr) {
                                        //是否为当前需要查找找到指定的变量赋值
                                        find = (local.equals(((NameExpr) node).getNameAsString()));
                                    } else {
                                        if (find && node instanceof IntegerLiteralExpr) {
                                            //若是则找到变量赋值的字面量
                                            int value = ((IntegerLiteralExpr) node).asNumber().intValue();
                                            returns.add(value);
                                        }
                                    }
                                }

                            }
                        }.visit(cu, null);


                    } else if (childNode instanceof IntegerLiteralExpr) {
                        //返回语法树直接返回字面量
                        int value = ((IntegerLiteralExpr) childNode).asNumber().intValue();
                        returns.add(value);


                    } else if (childNode instanceof FieldAccessExpr) {
                        //返回语法树使用了this取引用的变量，略

                    }
                }

            }
        }.visit(m1, null);

        System.out.println(returns);
    }
}



```


### 替换方法内容的示例

待替换的源代码
```java
package com.leaderli.litest;


import org.junit.Test;

import static org.junit.Assert.assertEquals;


class MyTest {
    @Test
    public void test() {
        String a = "123";
        assertEquals("123", a);
    }
}

```

```java

URI uri = Objects.requireNonNull(TestJavaParser.class.getResource("/")).toURI();
Path path = Paths.get(uri).getParent().getParent();
SourceRoot sourceRoot = new SourceRoot(path);
CompilationUnit cu = sourceRoot.parse("src/test/java/com/leaderli/litest", "MyTest.java");
//删除import
cu.walk(ImportDeclaration.class, e -> {
	if (e.isStatic() && "org.junit.Assert.assertEquals".equals(e.getNameAsString())) {
		e.remove();
	}
});


//删除方法调用
cu.walk(MethodCallExpr.class, e -> {
	e.getParentNode().get().remove();
});
Statement statement = StaticJavaParser.parseStatement("assert a.equals(\"123\");");
cu.walk(MethodDeclaration.class, e -> {
	e.getBody().get().addStatement(statement);

});
System.out.println(cu);
//将改动写回文件
sourceRoot.saveAll();
```
## API说明
`javaparser`使用访问者模式对语法树进行遍历访问，下面是一些常用的`visitor`

- `ReturnStmt` 含有`return`的语法树，例如`return 1;`
- `IntegerLiteralExpr`   int字面量，例如`int a = 1`中`1`即为`IntegerLiteralExpr`
- `VariableDeclarator` 变量声明，例如`int a = 1`，其中包含子语法树类型`SimpleName`，即为`a`
- `AssignExpr` 赋值语句，例如`a=2`，其中包含子语法树类型`NameExpr`,即为`a`


[[JavaParser_ Visited.pdf]]


