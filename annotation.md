
#注解 #反射 


## 元注解

`java.lang.annotation`提供了四种元注解，专门注解其他的注解（在自定义注解的时候，需要使用到元注解）：

### `Retention` 定义该注解的生命周期

1. `RetentionPolicy.SOURCE` : 在编译阶段丢弃。这些注解在编译结束之后就不再有任何意义，所以它们不会写入字节码。`@Override`, `@SuppressWarnings`都属于这类注解。
2. `RetentionPolicy.CLASS` : 在类加载的时候丢弃。在字节码文件的处理中有用。注解默认使用这种方式
3. `RetentionPolicy.RUNTIME` : 始终不会丢弃，运行期也保留该注解，因此可以使用反射机制读取该注解的信息。我们自定义的注解通常使用这种方式。

### `Target` 表示该注解用于什么地方。默认值为任何元素，表示该注解用于什么地方。可用的`ElementType`参数包括

1. `ElementType.CONSTRUCTOR`:用于描述构造器
2. `ElementType.FIELD`:成员变量、对象、属性（包括`enum`实例）​​
3. `ElementType.LOCAL_VARIABLE`:用于描述局部变量
4. `ElementType.METHOD`:用于描述方法
5. `ElementType.PACKAGE`:用于描述包
6. `ElementType.PARAMETER`:用于描述参数
7. `ElementType.TYPE`:用于描述类、接口(包括注解类型) 或`enum`声明

### `Documented` 一个简单的`Annotations`标记注解，表示是否将注解信息添加在`java`文档中

### `Inherited` 元注解是一个标记注解，`@Inherited`阐述了某个被标注的类型是被继承的。如果一个使用了`@Inherited`修饰的`annotation`类型被用于一个`class`，则这个`annotation`将被用于该`class`的子类

### `Repeatable` 用于标记该注解是否可重复使用，它需要制定另一个注解用以存放多个重复注解的值

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.PARAMETER, ElementType.TYPE})
@Repeatable(NotNull.NotNulls.class)
public @interface NotNull {

  String value();
  @Retention(RetentionPolicy.RUNTIME)
  @Target({ElementType.PARAMETER, ElementType.TYPE}
  @interface NotNulls {
    NotNull[] value();
  }
}
```

## 自定义注解

自定义注解类编写的一些规则:
`Annotation`型定义为`@interface`, 所有的`Annotation`会自动继承`java.lang.Annotation`这一接口,并且不能再去继承别的类或是接口.
参数成员只能用`public`或默认(`default`)这两个访问权修饰
参数成员只能用基本类型`byte`,`short`,`char`,`int`,`long`,`float`,`double`,`boolean`八种基本数据类型和`String`、`Enum`、`Class`、`annotations`等数据类型,以及这一些类型的数组.
要获取类方法和字段的注解信息，必须通过`Java`的反射技术来获取 `Annotation`对象,因为你除此之外没有别的获取注解对象的方法
注解也可以没有定义成员, 不过这样注解就没啥用了 PS:自定义注解需要使用到元注解

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.PARAMETER, ElementType.TYPE})
public @interface NotNull {
}
```

## 注解的原理

注解本质是一个继承了`Annotation`的特殊接口，其具体实现类是`Java`运行时生成的动态代理类。而我们通过反射获取注解时，返回的是`Java`运行时生成的动态代理对象`$Proxy1`。通过代理对象调用自定义注解（接口）的方法，会最终调用`AnnotationInvocationHandler`的`invoke`方法。该方法会从`memberValues`这个`Map`中索引出对应的值。而`memberValues`的来源是`Java`常量池。

```
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.PARAMETER, ElementType.TYPE})
public @interface MyAnnotation {

    String value() default "123";

}

@MyAnnotation
public class TestAnnotation {
    public static void main(String[] args) {
        //设置为true时，将会在项目跟目录生成代理类的源码
        System.setProperty("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");
        MyAnnotation myAnnotation = TestAnnotation.class.getAnnotation(MyAnnotation.class);
        Class<? extends MyAnnotation> proxy=myAnnotation.getClass();
        Class<? extends Annotation> annotationType= myAnnotation.annotationType();
        System.out.println("proxy = " + proxy);
        System.out.println("annotationType = " + annotationType);
        System.out.println(myAnnotation.value());
    }
}

```
当运行`main`方法时，会在根目录上生成一个代理类文件，通过反编译查看其代码如下
```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package com.sun.proxy;

import com.leaderli.liutil.MyAnnotation;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

public final class $Proxy1 extends Proxy implements MyAnnotation {
    private static Method m1;
    private static Method m2;
    private static Method m4;
    private static Method m0;
    private static Method m3;

    public $Proxy1(InvocationHandler var1) throws  {
        super(var1);
    }

    public final boolean equals(Object var1) throws  {
        try {
            return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final Class annotationType() throws  {
        try {
            return (Class)super.h.invoke(this, m4, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final int hashCode() throws  {
        try {
            return (Integer)super.h.invoke(this, m0, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final String value() throws  {
        try {
            return (String)super.h.invoke(this, m3, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m4 = Class.forName("com.leaderli.liutil.MyAnnotation").getMethod("annotationType");
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
            m3 = Class.forName("com.leaderli.liutil.MyAnnotation").getMethod("value");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}

```
main执行后的输出结果
>proxy = class com.sun.proxy.$Proxy1
>annotationType = interface com.leaderli.liutil.MyAnnotation
>123

通过查看`Class.getAnnotation`的源码我们可以发现，默认的JDK不会带上`sun`包下面的源码

```java
//1 获取注解的数据
annotationData().annotations.get(annotationClass);
//2 构建注解的数据
AnnotationData newAnnotationData = createAnnotationData(classRedefinedCount);
//3 已经是sun包下的类，无法看到源码，这个方法用于解析注解，这一步使用到字节码中常量池的索引解析，常量解析完毕会生成一个成员属性键值对作为下一个环节的入参
Map<Class<? extends Annotation>, Annotation> declaredAnnotations = AnnotationParser.parseAnnotations(getRawAnnotations(), getConstantPool(), this);

# //4 常量解析的部分代码片段，根据方法名，将方法的值缓存
for (int i = 0; i < numMembers; i++) {
    int memberNameIndex = buf.getShort() & 0xFFFF;
    String memberName = constPool.getUTF8At(memberNameIndex);
    Class<?> memberType = memberTypes.get(memberName);

    if (memberType == null) {
        // Member is no longer present in annotation type; ignore it
        skipMemberValue(buf);
    } else {
        Object value = parseMemberValue(memberType, buf, constPool, container);
        if (value instanceof AnnotationTypeMismatchExceptionProxy)
            ((AnnotationTypeMismatchExceptionProxy) value).
                setMember(type.members().get(memberName));
        memberValues.put(memberName, value);
    }
}
return annotationForMap(annotationClass, memberValues);

//5一个标准的JDK动态代理，而InvocationHandler的实例是AnnotationInvocationHandler，可以看它的成员变量、构造方法和实现InvocationHandler接口的invoke方法：
public static Annotation annotationForMap(final Class<? extends Annotation> type,
                                            final Map<String, Object> memberValues)
{
    return AccessController.doPrivileged(new PrivilegedAction<Annotation>() {
        public Annotation run() {
            return (Annotation) Proxy.newProxyInstance(
                type.getClassLoader(), new Class<?>[] { type },
                new AnnotationInvocationHandler(type, memberValues));
        }});
}


//节选AnnotationInvocationHandler代码

AnnotationInvocationHandler(Class<? extends Annotation> type, Map<String, Object> memberValues) {
    Class<?>[] superInterfaces = type.getInterfaces();
    if (!type.isAnnotation() ||
        superInterfaces.length != 1 ||
        superInterfaces[0] != java.lang.annotation.Annotation.class)
        throw new AnnotationFormatError("Attempt to create proxy for a non-annotation type.");
    this.type = type;
    this.memberValues = memberValues;
}
//可以看到除了默认的object方法外，属于自定义注解的方法的值，都是通过memberValues这个map来缓存结果的，结合生成的proxy动态类，可以发现
public Object invoke(Object proxy, Method method, Object[] args) {
    String member = method.getName();
    Class<?>[] paramTypes = method.getParameterTypes();

    // Handle Object and Annotation methods
    if (member.equals("equals") && paramTypes.length == 1 &&
        paramTypes[0] == Object.class)
        return equalsImpl(args[0]);
    if (paramTypes.length != 0)
        throw new AssertionError("Too many parameters for an annotation method");

    switch(member) {
    case "toString":
        return toStringImpl();
    case "hashCode":
        return hashCodeImpl();
    case "annotationType":
        return type;
    }

    // Handle annotation member accessors
    Object result = memberValues.get(member);

    if (result == null)
        throw new IncompleteAnnotationException(type, member);

    if (result instanceof ExceptionProxy)
        throw ((ExceptionProxy) result).generateException();

    if (result.getClass().isArray() && Array.getLength(result) != 0)
        result = cloneArray(result);

    return result;
}

```

所有 class 的 class，即 `Class` 类中，有关于 annotation 的属性`annotationData`,`annotationType`,因所有 class 都是`Class`的实例，所以所有 class 都会包含这些有关 annotaion 的属性。这就是为什么所有的 class 都可以使用`getAnnotations()`等方法


## 重复注解

```java
@Retention(RetentionPolicy.RUNTIME)
@Target( ElementType.TYPE)
@Repeatable()
public @interface Func{

  String value()

  @Retention(RetentionPolicy.RUNTIME)
  @Target( ElementType.TYPE)
  @interface Funcs{
    Func[] value();
  }
}


@Func("001")
@Func("002")
public class Bean{

}


```
获取重复注解的方式
```java

Bean.class.getAnnotation(Func.Funcs.class);
Bean.class.getAnnotationByType(Func.class);


//当不知道重复注解的类时，可使用如下方法

for(Annotation annotation:Bean.class.getAnnotations()){
  Method method = annotation.getClass().getMethod("value");
  Annotation[] finds = (Annotation[])method.invoke(annotation);
}
```

## 获取方法参数的注解

```java
Method method;

for(Anootation[] annotations:method.getParameterAnnotations()){
    
    //每个参数可以有多个注解
    System.out.println(Arrays.toString(annotations));

}
```
