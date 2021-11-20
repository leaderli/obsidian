[tutorial](http://www.javassist.org/tutorial/tutorial.html)
[tutorial-wiki](https://github.com/jboss-javassist/javassist/wiki/Tutorial-1)

## 快速入门
```maven
<dependency>
    <groupId>org.javassist</groupId>
    <artifactId>javassist</artifactId>
    <version>3.21.0-GA</version>
</dependency>
```

```java
package com.leaderli.liutil;

import javassist.*;
import javassist.bytecode.AnnotationsAttribute;
import javassist.bytecode.ConstPool;
import javassist.bytecode.annotation.Annotation;

import java.lang.reflect.Method;
import java.net.URL;
import java.net.URLClassLoader;
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;

public class TestAssist {

    public static final Map<String, Class<?>> ASSIST_CLASS = new HashMap<>();


    public static class ByteClassLoader extends URLClassLoader {

        public static final String PREFIX = "leaderli.com.";

        public ByteClassLoader(URL[] urls, ClassLoader parent) {
            super(urls, parent);
        }

        @Override
        protected Class<?> findClass(final String name) throws ClassNotFoundException {
            Class<?> cls = ASSIST_CLASS.get(name);
            if (cls != null) {
                return cls;
            }
            return super.findClass(name);
        }

    }

    static ByteClassLoader loader = new ByteClassLoader(new URL[]{}, ByteClassLoader.class.getClassLoader());

    static {
        try {
            init("Fuck");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void init(String name) throws Exception {
        ClassPool classPool = ClassPool.getDefault();

        String classname = ByteClassLoader.PREFIX + name;
        //生成类名
        CtClass ct = classPool.makeClass(classname);//Create class
        //生成属性
        CtField id = CtField.make("public int id;", ct);
        ConstPool constPool = ct.getClassFile().getConstPool();
        //属性增加注解
        AnnotationsAttribute annotationsAttribute = new AnnotationsAttribute(constPool, AnnotationsAttribute.visibleTag);
        Annotation annotation = new Annotation(Deprecated.class.getName(), constPool);
        annotationsAttribute.addAnnotation(annotation);
        id.getFieldInfo().addAttribute(annotationsAttribute);
        ct.addField(id);
        //生成构造器
        CtConstructor constructor = CtNewConstructor.make("public " + name + "(int pId){this.id=pId;}", ct);
        ct.addConstructor(constructor);
        //生成方法
        CtMethod helloM = CtNewMethod.make("public void hello2(String des){ System.out.println(des);}", ct);
        ct.addMethod(helloM);
        //方法增加注解
        helloM.getMethodInfo().addAttribute(annotationsAttribute);
        ASSIST_CLASS.put(name, ct.toClass());
    }

    public static void main(String[] args) throws Exception {


        Class<?> cls = loader.findClass("Fuck");
        System.out.println(cls);
        System.out.println(Arrays.toString(cls.getField("id").getAnnotations()));

        Object o = cls.getConstructor(int.class).newInstance(123);
        System.out.println(cls.getField("id").get(o));
        Method method = cls.getMethod("hello2", String.class);
        System.out.println(Arrays.toString(method.getAnnotations()));
        method.invoke(o, "fuck");
    }
}

```




## 概述
javassist是一个生成或修改java字节码的框架，他相对与[[ASM]]来说较轻量，使用起来较简洁，但有局限性。javassist不允许删除方法或字段，但允许更改名称。不允许修改方法的参数


### CtClass
java[[java字节码|字节码]]以二进制的形式存储在class文件中，在javassist中，类`CtClass`表示class的字节码对象，一个CtClass对象可以处理一个class字节码对象。

```java
ClassPool pool = ClassPool.getDefault();
CtClass cc = pool.get("test.Rectangle");
cc.setSuperclass(pool.get("test.Point"));
cc.writeFile();

//直接获取字节码文件
byte[] bytes = cc.toBytecode();
//使用当前线程的ClassLoader加载以CtClass对象转换的Class对象
Class clazz = cc.toClass();
```
在上面的例子中，修改了类的字节码对象，这项修改通过writeFile()将CtClass对象转换为[[java字节码|字节码]]文件并写到磁盘中。

#htext/red ==需要注意的是，CtClass做的修改是不会同步到已经被类加载器加载的类对象上的==

CtClass对象代表的class的文件中涉及到的其他class的引用，也需要在[[#ClassPool]]的[[#classpath]]中，

当不需要使用CtClass对象时，可以使用`detach()`方法，或者直接回收`ClassPool`，以避免加载过多Ctclass对象而造成巨大的内存消耗。调用detach之后，就不能调用这个CtClass对象的任何方法了，但可以调用[[#ClassPool]]的get方法，重新读取这个类文件，创建一个新的CtClass对象。

也可以直接回收[[#ClassPool]]，或者创建一个新的[[#ClassPool]]
```java
CtClass cc;
//回收
cc.detach();
```

### ClassPool
ClassPool是CtClass对象的容器。它按需要读取类文件来构造CtClass对象，并且保存CtClass对象以便以后使用。

ClassPool是一个存储CtClass的Hash表，ClassPool的get()函数用于从Hash表中查找指定Key的CtClass对象，如果没有找到，get()函数会在其[[#classpath]]下查找指定名称的[[java字节码|字节码]]文件，创建并返回一个新的CtClass对象，这个新对象会保存在Hash表中。

如果程序在web应用服务器上运行，则可能需要创建多个ClassPool实例。正确的做法应该为每一个ClassLoader创建一个ClassPool实例，ClassPool应当通过构造函数来生成，而不是调用getDefault()来创建。


#htext/red ==ClassPool的层级关系应当保持和ClassLoader一致==


```java
//parent classloader
ClassPool parent = ClassPool.getDefault();

//child classloader
ClassPool child = new ClassPool(parent);


```
## 修改与加载类

对于已经被ClassLoader加载的类，CtClass对其[[java字节码|字节码]]做的修改是无法及时生效的。

```java
//com.AssistDemo.java
package com;  
public class AssistDemo {  
  
    private int id = 1;  
}
```

```java
ClassPool pool = ClassPool.getDefault();  
CtClass cc = pool.get(AssistDemo.class.getName());  
cc.toClass();
```
上述代码会抛出如下异常

```java
javassist.CannotCompileException: by java.lang.LinkageError:
loader (instance of  sun/misc/Launcher$AppClassLoader):
attempted  duplicate class definition for name: "com/AssistDemo"
```

当我们使用如下方式去修改时，则可以生效

```java
CtClass cc = pool.get("com.AssistDemo");  
  
CtField id = cc.getField("id");  
id.setName("newName");  
  
Class<?> aClass = cc.toClass();  
  
System.out.println(Arrays.toString(aClass.getDeclaredFields()));  
System.out.println(Arrays.toString(AssistDemo.class.getDeclaredFields()));

```
>[private int com.AssistDemo.newName]
>[private int com.AssistDemo.newName]


如果事先知道需要修改哪些类，修改类的最简方法如下

1. 调用ClassPool.get() 获取CtClass对象
2. 修改CtClass
3. 调用CtClass对象的writeFile()或者toBytecode()获取修改过的类

#htext/red==如果在加载时，可以确定是否需要修改某个类，用户必须使jvassist和类加载器协作，以便在类加载过程中先修改字节码。用户可以定义自己的类加载器，也可以使用javassist提供的类加载器==


[[#类加载器]]

如果一个CtClass对象通过writeFile()，toClass()，toBytecode()被转换成一个类文件，此CtClass对象会被冻结起来，不允许再修改。因为一个类只能被JVM加载一次。

但是，一个被冻结的CtClass也可以解冻，解冻后则可以继续修改了。

```java

CtClass cc = ...;

cc.writeFile();
cc.defrost();
cc.setSuperClass(...)

```


## classpath

如果程序运行在tomcat等web服务器上，ClassPool可能无法找到用户的类，因为web服务器使用多个类加载器作为系统类加载器，在这种情况下，ClassPool必须添加额外的类搜索路径。
```java
//获取使用JVM的类搜索路径
ClassPool classPool = ClassPool.getDefault();  
//将this指向的类的classloader的类加载路径添加到pool的类加载路径
classPool.insertClassPath(new ClassClassPath(this.getClass()));

```

也可以注册一个目录作为类搜索路径

```java
ClassPool classPool = ClassPool.getDefault();  
classPool.insertClassPath("/user/local/javalib");
```

也可以直接传递byte数组

```java
ClassPool cp = ClassPool.getDefault();
//java byte ;
byte[] b;
//className
String name;
cp.insertClassPath(new ByteArrayClassPath(name, b));
CtClass cc = cp.get(name);
```

当不知道类名的时候可以使用输入流

```java
ClassPool cp = ClassPool.getDefault();
//字节码文件流
InputStream ins;
CtClass cc = cp.makeClass(ins);
```


##  类加载器

注入当前线程的上下文类加载器
```java
try {
    Class.forName("MyClass");
} catch(ClassNotFoundException e) {
    ClassPool pool = ClassPool.getDefault();
    CtClass cc = pool.makeClass("MyClass");
    cc.toClass(this.getClass().getClassLoader(), this.getClass().getProtectionDomain());
    Class.forName("MyClass");
}
```

当在tomcat等容器上运行时，toClass() 使用的上下文类加载器可能是不合适的，

```java
//需要操作的类对象
CtClass cc;
//传递合适的类加载器
Class c = cc.toClass(bean.getClass().getClassLoader());
```

javaassist提供一个类加载器javassist.Loader。它使用javaassist.ClassPool对象来读取类文件。

例如
```java
ClassPool pool = ClassPool.getDefault();  
CtClass cc = pool.get("com.AssistDemo");  
  
Loader cl = new Loader(pool);  
Class<?> aClass = cl.loadClass("com.AssistDemo");  
Object o = aClass.newInstance();  
System.out.println(aClass + "@" + aClass.hashCode()+" "+aClass.getClassLoader());  
aClass = AssistDemo.class;  
System.out.println(aClass + "@" + aClass.hashCode()+" "+aClass.getClassLoader());

//class com.AssistDemo@425918570 javassist.Loader@368102c8
//class com.AssistDemo@1100439041 sun.misc.Launcher$AppClassLoader@18b4aac2
```


如果希望在加载时按需修改类，则可以向javassist.Loader添加一个监听器

```java
public interface Translator {
    public void start(ClassPool pool) throws NotFoundException, CannotCompileException;
    public void onLoad(ClassPool pool, String classname) throws NotFoundException, CannotCompileException;
}
```

注意，onLoad() 不必调用 toBytecode() 或 writeFile()，因为 javassist.Loader 会调用这些方法来获取类文件。

使用示例

```java
class MyTranslator implements Translator {  
  
    @Override  
 public void start(ClassPool classPool) throws NotFoundException, CannotCompileException {  
  
        System.out.println("start:" + classPool);  
    }  
  
    @Override  
 public void onLoad(ClassPool classPool, String name) throws NotFoundException, CannotCompileException {  
        System.out.println("onload:" + classPool + " " + name);  
  
        CtClass ctClass = classPool.get(name);  
        System.out.println(Arrays.toString(ctClass.toClass().getDeclaredFields()));  
  
        ctClass.defrost();  
        ctClass.getField("id").setName("fuck");  
  
    }  
}

```

```java
public void test4() throws Throwable {  
  
    ClassPool pool = ClassPool.getDefault();  
    Loader cl = new Loader(pool);  
    String name = "com.AssistDemo";  
    cl.addTranslator(pool, new MyTranslator());  
    System.out.println(Arrays.toString(cl.loadClass(name).getDeclaredFields()));  
  
}
```
>start:[class path: java.lang.Object.class;]
onload:[class path: java.lang.Object.class;] com.AssistDemo
[private int com.AssistDemo.id]
[private int com.AssistDemo.fuck]



## 注意事项

Java 中的装箱和拆箱是语法糖。没有用于装箱或拆箱的字节码。所以 Javassist 的编译器不支持它们

```java
//无法通过编译
CtField id = CtField.make("public Integer id = 1;", ct);

//需要显式的装箱拆箱
CtField id = CtField.make("public Integer id = new Integer(1);", ct);
```


可变参数，泛型，类型转换等语法糖，都会遇到类似问题
## 常用API
### 定义新类

```java
ClassPool pool = ClassPool.getDefault();
CtClass cc = pool.makeClass("Point");

//定义接口
pool.makeInterface("IPoint");
```


### 添加注解

```java
//添加枚举类型的注解

AnnotationsAttribute attr = new AnnotationsAttribute(constpool, AnnotationsAttribute.visibleTag);

Annotation annot = new Annotation("org.mengyun.tcctransaction.api.Compensable", constpool);
EnumMemberValue enumMemberValue = new EnumMemberValue(constpool);
enumMemberValue.setType("org.mengyun.tcctransaction.api.Propagation");
enumMemberValue.setValue("SUPPORTS");
annot.addMemberValue("propagation", enumMemberValue);
annot.addMemberValue("confirmMethod", new StringMemberValue(ctMethod.getName(), constpool));

```
需要特别注意的是，注解的值，需要使用特殊的方法赋值，不能直接赋值

```java
ConstPool constPool = ct.getClassFile().getConstPool();

//方法增加注解
AnnotationsAttribute annotationsAttribute = new AnnotationsAttribute(constPool, AnnotationsAttribute.visibleTag);

Annotation requestMapping = new Annotation(RequestMapping.class.getName(), constPool);
StringMemberValue url = new StringMemberValue(constPool);
url.setValue("/6666");

EnumMemberValue type = new EnumMemberValue(constPool);
type.setType(PackageType.class.getName());
type.setValue(PackageType.TOUDA_JSON_OUT.name());
requestMapping.addMemberValue("url", url);
requestMapping.addMemberValue("type", type);

annotationsAttribute.addAnnotation(requestMapping);

helloM.getMethodInfo().addAttribute(annotationsAttribute);

```



### 修改成员变量名称

```java
ClassPool pool = ClassPool.getDefault();  
CtClass ctClass = pool.get("com.AssistDemo");  
CtField id = ctClass.getField("id");  
CodeConverter codeConverter = new CodeConverter();  
codeConverter.redirectFieldAccess(id,ctClass,"id2");  
id.setName("id2");  
ctClass.instrument(codeConverter);  
ctClass.toClass().newInstance();
```

上述方法不会将成员变量id删除，而是新生成一个成员变量id2，以及替换所有使用到成员变量id的地方。