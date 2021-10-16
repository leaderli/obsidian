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