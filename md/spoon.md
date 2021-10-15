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
```