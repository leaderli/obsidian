# javassist

javassist是一个生成java字节码的框架，他相对来说比较简单。
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

## 添加注解

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
需要特别注意的是,注解的值，需要使用特殊的方法赋值，不能直接赋值

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

## 加入当前classloader

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