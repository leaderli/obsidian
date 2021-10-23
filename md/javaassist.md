# javassist

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
javassist是一个生成或修改java字节码的框架，他相对与[[ASM]]来说较轻量，使用起来较简洁，但有局限性

java[[java字节码|字节码]]以二进制的形式存储在class文件中，在javassist中，类`CtClass`表示class文件，一个CtClass对象可以处理一个class文件。

```java
ClassPool pool = ClassPool.getDefault();
CtClass cc = pool.get("test.Rectangle");
cc.setSuperclass(pool.get("test.Point"));
cc.writeFile();
```

### ClassPool
ClassPool是CtClass对象的容器。它按需要读取类文件来构造CtClass对象，并且保存CtClass对象以便以后使用

## 冻结修改

如果一个CtClass对象通过writeFile()，toClass()，toBytecode()被转换称一个类文件，此CtClass对象会被冻结起来，不允许再修改。因为一个类只能被JVM加载一次。

但是，一个被冻结的CtClass也可以解冻

```java

CtClass cc = ...;

cc.writeFile();
cc.defrost();
cc.setSuperClass(...)

```


## 类搜索路径

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

或者使用输入流

```java
ClassPool cp = ClassPool.getDefault();
//字节码文件流
InputStream ins;
CtClass cc = cp.makeClass(ins);
```


##   ClassPool

ClassPool在执行期间，必须包含所有使用到的CtClass实例，当不需要使用CtClass对象时，可以使用`detach()`方法，及时清理`ClassPool`，或者直接回收`ClassPool`

```java
CtClass cc;

cc.writeFile();

//回收
cc.detach();
```


## 常用示例
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

### 加入当前classloader

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