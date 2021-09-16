#java #反射

## 概述

```ad-li
JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制
```

## Class类

每个java类运行时都在JVM里表现为一个class对象，基本类型`boolean`，`byte`，`char`，`short`，`int`，`long`，`float`，`double`和关键字`void`同样表现为 class 对象

```java
public final class Class<T> implements java.io.Serializable,
                              GenericDeclaration,
                              Type,
                              AnnotatedElement {
    private static final int ANNOTATION= 0x00002000;
    private static final int ENUM      = 0x00004000;
    private static final int SYNTHETIC = 0x00001000;

    private static native void registerNatives();
    static {
        registerNatives();
    }

    /*
     * Private constructor. Only the Java Virtual Machine creates Class objects.   //私有构造器，只有JVM才能调用创建Class对象
     * This constructor is not used and prevents the default constructor being
     * generated.
     */
    private Class(ClassLoader loader) {
        // Initialize final field for classLoader.  The initialization value of non-null
        // prevents future JIT optimizations from assuming this final field is null.
        classLoader = loader;
    }

```

- Class也是类的一种
- 编写的类被编译后产生一个Class对象，其表示创建的类的类型信息，而这个Class对象保存在同名class文件中（字节码文件）
- 每个通过关键字class标识的类，在内存中有且仅有一个与之对应的Class对象来描述其类型信息
- Class类只存在私有构造函数，因此对应Class对象只有JVM创建和加载
- Class类的对象在运行时提供获得某个对象的类型信息，成员变量，方法，构造方法，注解等。

## 类加载
1. [[java类加载机制]]
2. [[java字节码]]

## 常用API
### Class类
| 方法              | 说明                                     |
| ----------------- | ---------------------------------------- |
| `getName()`       | 取全限定的类名(包括包名)，即类的完整名字 | 
| `getSimpleName()` | 获取类名(不包括包名)                     |
## 可变参数方法的反射

```java
public static void me(Object ... objects){
    for (Object object : objects) {
        System.out.println(object);
    }
}
@Test
public void  test() throws Exception {
    Class clazz = this.getClass();
    //Method method = clazz.getMethod("me",(new Object[0]).getClass());
    //Method method = clazz.getMethod("me",Array.newInstance(Object.class,0).getClass());
    Method method = clazz.getMethod("me",Class.forName("[Ljava.lang.Object;"));
    //1
    Object objs = Array.newInstance(object.class,2);
    Array.set(objs,0,1);
    Array.set(objs,1,"test");
    method.invoke(clazz,objs);
    //2
    Object[] obj = {1,"test"}
    method.invoke(clazz,new Object[]{obj});
    

    //例如对于String的可变参数可通过如下方式去调用
    Object[] arr = (Object[])Array.newInstance(String.class, 5);
    Array.set(arr, 0, "5");
    Array.set(arr, 1, "1");
    Array.set(arr, 2, "9");
    Array.set(arr, 3, "3");
    Array.set(arr, 4, "7");
    
    method.invoke(clazz,new Object[]{arr});
}
```

可变参数不可直接显式使用 null 作为参数

```java
public class TestStatic {
    public static void main(String[] args) {
        String s = null;
        m1(s);
        Util.log("begin null");
        m1(null);
    }

    private static void m1(String... strs) {
        System.out.println(strs.length);
    }

}
```

```java
0: aconst_null          //将null压入操作栈
1: astore_1             //弹出栈顶(null)存储到本地变量1
2: iconst_1             //压栈1此时已经到方法m1了，在初始化参数，此值作为数组长度
3: anewarray     #2     //新建数组            // class java/lang/String
6: dup                  //复制数组指针引用
7: iconst_0             //压栈0，作为数组0角标
8: aload_1              //取本地变量1值压栈，作为数组0的值
9: aastore              //根据栈顶的引用型数值（value）、数组下标（index）、数组引用（arrayref）出栈，将数值存入对应的数组元素中
10: invokestatic  #3    //此时实际传递的是一个数组，只是0位置为null的元素 Method m1:([Ljava/lang/String;)V
13: iconst_1
14: anewarray     #4    //class java/lang/Object
17: dup
18: iconst_0
19: ldc           #5    //String begin null
1: aastore
22: invokestatic  #6    //Method li/Util.log:([Ljava/lang/Object;)V
25: aconst_null         //此处并没有新建数组操作，直接压栈null
26: invokestatic  #3    //此处一定会抛出空指针  Method m1:([Ljava/lang/String;)V
29: return
```

## 反射工具类创建数组
```java
int arr[] = (int[])Array.newInstance(int.class, 5);
Array.set(arr, 0, 5);
Array.set(arr, 1, 1);
Array.set(arr, 2, 9);
Array.set(arr, 3, 3);
Array.set(arr, 4, 7);
System.out.print("The array elements are: ");
for(int i: arr) {
    System.out.print(i + " ");
}
```
