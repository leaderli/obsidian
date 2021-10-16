一个用来分析，重写，转换java源代码的库，将java源文件编译成[[AST]]。

[官方文档](https://spoon.gforge.inria.fr/)


## 安装

```xml
<dependency>
  <groupId>fr.inria.gforge.spoon</groupId>
  <artifactId>spoon-core</artifactId>
  <version>9.1.0</version>
</dependency>
```

## 快速使用

```java
CtClass<?> l = Launcher.parseClass("public class A { public void m() { System.out.println(\"yeah\");} }");  
  
l.setLabel("fuck");  
System.out.println(l.toStringWithImports());  
Object o = l.newInstance();  
Method m = o.getClass().getDeclaredMethod("m");  
m.invoke(o);
```

## 用于代码校验

例如，我们不允许使用`new TreeSet()`
```java
@Test
void noTreeSet() throws Exception {
	SpoonAPI spoon = new Launcher();
	spoon.addInputResource("src/main/java/");
	spoon.buildModel();
	
	assertEquals(0, spoon.getFactory().Package().getRootPackage().getElements(new AbstractFilter<CtConstructorCall>() {
		@Override
		public boolean matches(CtConstructorCall element) {
			return element.getType().getActualClass().equals(TreeSet.class);
		};
	}).size());
}

```

强制所有junit类，必须以`Test`开头或结尾

```java
SpoonAPI spoon = new Launcher();  
spoon.addInputResource("src/test/java/");  
spoon.buildModel();  
  
List<CtMethod<?>> methods = spoon.getModel().getRootPackage().getElements(new TypeFilter<CtMethod<?>>(CtMethod.class) {  
    @Override  
 public boolean matches(CtMethod<?> element) {  
        return super.matches(element) && element.getAnnotation(Test.class) != null;  
    }  
});  
for (CtMethod<?> ctMethod : methods) {  
    String simpleName = ctMethod.getParent(CtClass.class).getSimpleName();  
    assertTrue("naming contract violated for " + simpleName, simpleName.startsWith("Test") || simpleName.endsWith("Test"));  
}
```

校验public方法，必须有大于20字以上的注释
```java
SpoonAPI spoon = new Launcher();  
spoon.addInputResource("src/test/java/");  
spoon.buildModel();  
List<String> notDocumented = new ArrayList<>();  
for (CtMethod<?> method : spoon.getModel().getElements(new TypeFilter<>(CtMethod.class))) {  
    if (method.hasModifier(ModifierKind.PUBLIC) && method.getTopDefinitions().size() == 0) {  
        if (method.getDocComment().length() < 20) { // at least 20 characters  
 notDocumented.add(method.getParent(CtType.class).getQualifiedName() + "#" + method.getSignature());  
        }  
    }  
}  
if (notDocumented.size() > 0) {  
    fail(notDocumented.size() + " public methods should be documented with proper API documentation: \n" + StringUtils.join(notDocumented, "\n"));  
}
```


查找所有未被使用的私有成员变量
```java
SpoonAPI spoon = new Launcher();  
spoon.addInputResource("src/test/java/");  
CtModel model = spoon.buildModel();  
// 查找所有成员变量  
List<CtField<?>> fields = model.getElements(new TypeFilter<>(CtField.class));  
// 移除非私有成员变量，移除序列化成员变量  
fields.removeIf(v -> !v.isPrivate());  
fields.removeIf(v -> v.getSimpleName().equals("serialVersionUID"));  
// Step 4: Query the model for all fieldReads  
// 查找所有使用成员变量的模型  
List<CtFieldRead<?>> fieldRead = model.getElements(new TypeFilter<>(CtFieldRead.class));  
// 移除没有变量声明的模型  
fieldRead.removeIf(v -> v.getVariable().getFieldDeclaration() == null);  
// 记录被使用的成员变量  
HashSet<CtField<?>> lookUp = fieldRead.stream()  
        .map(CtFieldRead::getVariable)  
        .map(CtFieldReference::getFieldDeclaration)  
        .collect(Collectors.toCollection(HashSet::new));  
// 查找未被使用的成员变量  
List<CtField<?>> fieldsWithRead = fields.stream()  
        //      every field must have a read  
 .filter(field -> !lookUp.contains(field))  
        .collect(Collectors.toList());  
assertEquals("Some Fields have no read/write", Collections.emptyList(), fields);
```

## 打印

在没有冲突的情况下，将使用到的类，以`import`的方式导入
```java
launcher.getEnvironment().setAutoImports(true);
```


```java
SpoonAPI spoon = new Launcher();  
spoon.getEnvironment().setAutoImports(true);  
spoon.addInputResource("src/test/java/");  
CtModel model = spoon.buildModel();  
CtType<?> ctType = model.getElements(new TypeFilter<CtType<?>>(CtType.class) {  
    @Override  
 public boolean matches(CtType<?> element) {  
  
        return super.matches(element);  
    }  
}).get(0);  
System.out.println(ctType.toString());  
//使用全限定名打印
System.out.println(ctType.toStringDebug());  
//会将package和import同时打印出来
System.out.println(ctType.toStringWithImports());
```



## 查找指定元素

查找某个类的所有方法
```java
methods = ctClass.getMethods();
```

可以通过[`CtRole`](https://spoon.gforge.inria.fr/mvnsites/spoon-core/apidocs/spoon/reflect/path/CtRole.html)来查找指定角色的子元素

```java
methods = ctClass.getValueByRole(CtRole.METHOD);
```

或者查找所有子元素
```java
allDescendants = ctElement.getDirectChildren();
```


也可以通过过滤器来查找满足条件的元素

例如：
```java
//查找所有赋值语句
list1 = methodBody.getElements(new TypeFilter(CtAssignment.class));

//所有被deprecated注解的类
list2 = rootPackage.getElements(new AnnotationFilter(Deprecated.class));


//也可以指定过滤的具体条件
list3 = rootPackage.filterChildren(
  new AbstractFilter<CtField>(CtField.class) {
    @Override
    public boolean matches(CtField field) {
      return field.getModifiers.contains(ModifierKind.PUBLIC);
    }
  }
).list();

```

通过迭代器的方式遍历子元素

```java
CtIterator iterator = new CtIterator(root);
while (iterator.hasNext()) {
	CtElement el = iterator.next();
	//do something on each child of root
}
```

`CtBFSIterator`和`CtIterator`相似，但是广度优先


## 创建AST

```java
Launcher launcher = new Launcher();  
launcher.getEnvironment().setAutoImports(true);  
Factory factory = launcher.getFactory();  
CtClass<Object> fuck = factory.Class().create("Fuck");  
fuck.addModifier(ModifierKind.PUBLIC);  
  
CtTypeReference<Date> dateRef = factory.Code().createCtTypeReference(Date.class);  
CtTypeReference<List<Date>> listRef = factory.Code().createCtTypeReference(List.class);  
  
listRef.addActualTypeArgument(dateRef);  
  
  
CtField<List<Date>> listOfDates = factory.createField();  
listOfDates.setSimpleName("dates");  
listOfDates.setType(listRef);  
listOfDates.addModifier(ModifierKind.PRIVATE);  
  
fuck.addField(listOfDates);  
  
  
System.out.println(fuck.toStringWithImports());  
Object o = fuck.newInstance();  
System.out.println(o);
```

```jva
import java.util.Date;
import java.util.List;
public class Fuck {
    private List<Date> dates;
}
Fuck@175c2241
```


## spoon的AST元素类型

![[Pasted image 20211016233242.png]]

### CtArrayRead
```java
    int[] array = new int[10];
    System.out.println(
    array[0] // <-- array read
    );
```

### CtArrayWrite
```java
    Object[] array = new Object[10];
    // array write
    array[0] = "new value";
```

### CtAssert
```java
assert 1+1==2
```

### CtAssignment
```java
    int x;
    x = 4; // <-- an assignment
```
### CtBinaryOperator
```java
    // 3+4 is the binary expression
    int x = 3 + 4;
```

### CtBlock
```java
 { // <-- block start
  System.out.println("foo");
 }
	
```

### CtBreak
```java
    for(int i=0; i<10; i++) {
        if (i>3) {
				break; // <-- break statement
        }
    }
```

### CtCase
```java
int x = 0;
switch(x) {
    case 1: // <-- case statement
      System.out.println("foo");
}
```

### CtConditional
```java
    System.out.println(
       1==0 ? "foo" : "bar" // <-- ternary conditional
    );

```

### CtConstructorCall
```java
    new Object();
```

### CtContinue
```java
    for(int i=0; i<10; i++) {
        if (i>3) {
				continue; // <-- continue statement
        }
    }
```

### CtDo
```java
    int x = 0;
    do {
        x=x+1;
    } while (x<10);
```

### CtExecutableReferenceExpression
```java
    java.util.function.Supplier p =
      Object::new;
```

### CtFieldRead
```java
    class Foo { int field; }
    Foo x = new Foo();
    System.out.println(x.field);
```

### CtFieldWrite
```java
    class Foo { int field; }
    Foo x = new Foo();
    x.field = 0;
```

### CtFor
```java
    // a for statement
    for(int i=0; i<10; i++) {
    	System.out.println("foo");
    }

```

### CtForEach
```java
    java.util.List l = new java.util.ArrayList();
    for(Object o : l) { // <-- foreach loop
    	System.out.println(o);
    }
```

### CtIf
```java
    if (1==0) {
    	System.out.println("foo");
    } else {
    	System.out.println("bar");
    }

```

### CtInvocation
```java
    // invocation of method println
    // the target is "System.out"
    System.out.println("foo");

```

### CtJavaDoc
```java

/**
 * Description
 * @tag a tag in the javadoc
*/

```

### CtLambda
```java

    java.util.List l = new java.util.ArrayList();
    l.stream().map(
      x -> { return x.toString(); } // a lambda
    );

```

### CtLiteral
```java
    int x = 4; // 4 is a literal
```

### CtLocalVariable
```java
    // defines a local variable x
    int x = 0;


    // local variable in Java 10
    var x = 0;
```

### CtNewArray
```java
    // inline creation of array content
    int[] x = new int[] { 0, 1, 42}
```

### CtNewClass
```java
   // an anonymous class creation
   Runnable r = new Runnable() {
    	@Override
    	public void run() {
    	  System.out.println("foo");
    	}
   };

```

### CtOperatorAssignment
```java
    int x = 0;
    x *= 3; // <-- a CtOperatorAssignment
```

### CtReturn
```java
   Runnable r = new Runnable() {
    	@Override
    	public void run() {
    	  return; // <-- CtReturn statement
    	}
   };
```

### CtSuperAccess
```java
    class Foo { int foo() { return 42;}};
    class Bar extends Foo {
    int foo() {
      return super.foo(); // <-- access to super
    }
    };

```

### CtSwitch
```java
int x = 0;
switch(x) { // <-- switch statement
    case 1:
      System.out.println("foo");
}
```

### CtSwitchExpression
```java
int i = 0;
int x = switch(i) { // <-- switch expression
    case 1 -> 10;
    case 2 -> 20;
    default -> 30;
};
```

### CtSynchronized
```java
   java.util.List l = new java.util.ArrayList();
   synchronized(l) {
    	System.out.println("foo");
   }

```

### CtTextBlock
```java

    String example = """
		      Test String
       	      """;
```

### CtThisAccess
```java
    class Foo {
    int value = 42;
    int foo() {
      return this.value; // <-- access to this
    }
    };

```

### CtThrow
```java
    throw new RuntimeException("oops")
```

### CtTry
```java
    try {
    	System.out.println("foo");
    } catch (Exception ignore) {}

```

### CtTryWithResource
```java
   // br is the resource
   try (java.io.BufferedReader br = new java.io.BufferedReader(new java.io.FileReader("/foo"))) {
   	br.readLine();
  }
```

### CtTypeAccess
```java
    // access to static field
    java.io.PrintStream ps = System.out;


    // call to static method
    Class.forName("Foo")

    // method reference
    java.util.function.Supplier p =
      Object::new;


    // instanceof test
    boolean x = new Object() instanceof Integer // Integer is represented as an access to type Integer


    // fake field "class"
    Class x = Number.class

```

### CtTypePattern

```java
    Object obj = null;
    boolean longerThanTwo = false;
    // String s is the type pattern, declaring a local variable
    if (obj instanceof String s) {
        longerThanTwo = s.length() > 2;
    }

```

### CtUnaryOperator
```java
    int x=3;
    --x; // <-- unary --
```

### CtVariableRead
```java
    String variable = "";
    System.out.println(
      variable // <-- a variable read
    );

```

### CtVariableWrite
```java
    String variable = "";
    variable = "new value"; // variable write


    String variable = "";
    variable += "";
```

### CtWhile
```java
    int x = 0;
    while (x!=10) {
        x=x+1;
    };
```

### CtYieldStatement

```java
    int x = 0;
    x = switch ("foo") {
        default -> {
					x=x+1;
					yield x; //<--- yield statement
					}
    };


    int x = 0;
    x = switch ("foo") {
        default -> 4; //<---  implicit yield statement
    };

```

### CtAnnotation
```java
    // statement annotated by annotation @SuppressWarnings
    @SuppressWarnings("unchecked")
    java.util.List<?> x = new java.util.ArrayList<>()
```

### CtClass
```java
    // a class definition
    class Foo {
       int x;
    }

```

### CtInterface
```java
    // an interface definition
    interface Foo {
       void bar();
    }
```

### Class comment
```java
// class comment
class A {
  // class comment
}
```


### CtComment
-   File comments (comment at the beginning of the file, generally licence) `CtComment.CommentType.FILE`
-   Line comments (from `//`to end line) `CtComment.CommentType.INLINE`
-   Block comments (from `/* to */`) `CtComment.CommentType.BLOCK`
-   Javadoc comments (`from /** to */`) `CtComment.CommentType.JAVADOC`