---
title: genericsAndReflect
date: 2019-10-19 19:51:27
categories: java
tags:
  - 泛型
  - 反射
---

## 泛型 `extends super`


<div class="li_hl">
    不管是extends或是super，只能使用在变量声明上，实际赋值的时候，一定是指定具体实现类的，这就是extend和super的方法会有所限制
</div>

```java

//那么对于<? extends T>来说，具体的实现类的泛型A只是变量声明的泛型T的子类，如果以T进行插入时，是无法保证插入的class类型，一定是A，所以extends禁用插入动作
//这个时候，从fruits取出的，其实是Apple泛型的元素，因此一定是Fruit类，但如果像fruits插入一个Banana泛型的元素，则与实际的泛型类Apple冲突，所以禁止使用fruits这个变量去进行插入动作
List<Apple> apples = new ArrayList<Apple>();
List<? extends Fruit> fruits = apples;

// 对于<? super T>来说，具体的实现类的泛型A一定是变量声明的泛型T的父类，如果以T类型进入取值操作，无法保证取出的值一定是T类型，因为A一定是T的父类，所以插入的所有实例一定也是A的多态
//这个时候，若apples中插入一个Apple泛型的元素，因为Apple是继承自Fruit泛型的子类，所以是可以插入到fruits中的，但是如果从apples取一个元素，因为fruits中可能有Banana泛型的元素，其余apples的实际泛型是冲突的，所以禁止使用取值操作。
List<Fruit> fruits = new ArrayList<Fruit>();
List<? super Apple> apples = fruits;
```

作为方法参数时，对于一个典型的示例来说，一般 mapper 函数是这样声明的

```java
 <R> Stream<R> map(Function<? super T, ? extends R> mapper);

 //上述形参的泛型表示实际mapper方法调用时是使用如下泛型的
R apply(T t);
```

其含义是，当我们调用这个 mapper 的 apply 时，我们的请求参数为 T 类型，其返回值为 R 类型。
所以在当我们在 apply 方法内部，当我调用 T 类型的父类方法是可以接受的，我实际传递 mapper 时，可以直接使用泛型为 T 的父类的形参。而 apply 方法的返回值，我返回 R 类型的子类也是可以接受的，所以传递 mapper 时也可以直接使用泛型为 R 的子类的返回类型

## 泛型的应用

```java
class Node<T extends Node<? extends Node<?>>> {

 T parent;
}
class RootNode extends Node<Node<?>>{

}

class Node1 extends Node<RootNode>{

}
class Node2 extends Node<RootNode>{

}

//在指定T类型时，其实R类型已经固定了
class B<T extends Node<R>, R extends Node<?>> {

 T node;
 R parent;

}

//C1 会直接提示错误
class C1 extends B<Node1,Node2>{

}
class C2 extends B<Node1,RootNode>{

}

```

## 桥接方法

当类定义中的类型参数没有任何限制时，在类型擦除中直接被替换为Object，即形如<T>和<?>的类型参数都被替换为Object。，

```java
public class Node<T> {

    public T data;

    public Node(T data) { this.data = data; }

    public void setData(T data) {
        System.out.println("Node.setData");
        this.data = data;
    }
}

public class MyNode extends Node<Integer> {
    public MyNode(Integer data) { super(data); }

    public void setData(Integer data) {
        System.out.println("MyNode.setData");
        super.setData(data);
    }
}
```

当做如下使用时

```java
Node node = new MyNode(5);
n.setData("Hello");
```

当子类重写了`setData`方法时其参数为`Integer`，我们的子类中实际是没有 `setData(Object.class)`的方法的，`java`编译器在进行类型擦除的时候会自动生成一个`synthetic`方法，也叫`bridge`方法,我们通过生成的字节码可以看到实际`bridge`方法，首先校验类型是否为`Integer`，然后在调用`setData(Integer.class)`因此，上述代码会抛出`ClassCastException`

```java
//通过javap -c 的方法可以显示桥接方法
public void setData(java.lang.Integer);
    descriptor: (Ljava/lang/Integer;)V
    flags: ACC_PUBLIC
    Code:
        stack=2, locals=2, args_size=2
            0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
            3: ldc           #3                  // String MyNode.setNode
            5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
            8: aload_0
            9: aload_1
            10: invokespecial #5                  // Method com/li/springboot/bridge/Node.setData:(Ljava/lang/Object;)V
            13: return
        LineNumberTable:
        line 11: 0
        line 12: 8
        line 13: 13
        LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      14     0  this   Lcom/li/springboot/bridge/MyNode;
            0      14     1 integer   Ljava/lang/Integer;
    MethodParameters:
        Name                           Flags
        integer


public void setData(java.lang.Object);
    descriptor: (Ljava/lang/Object;)V
    flags: ACC_PUBLIC, ACC_BRIDGE, ACC_SYNTHETIC
    Code:
        stack=2, locals=2, args_size=2
            0: aload_0
            1: aload_1
            2: checkcast     #11                 // class java/lang/Integer
            5: invokevirtual #12                 // Method setData:(Ljava/lang/Integer;)V
            8: return
        LineNumberTable:
        line 3: 0
        LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  this   Lcom/li/springboot/bridge/MyNode;
    MethodParameters:
        Name                           Flags
        integer                        synthetic

```


## 子类继承时不使用泛型

```java
public interface Father<T> {
  void func(T t);
}

public Son implements Father<String> {

  public void func(String t){
    ...
  }
}
```
##  当使用通配符不能直接使用时，可使用强转来实现

```java
   @SuppressWarnings("unchecked")
    public <T extends EventObject> List<ILiEventListener<T>> get(Class<T> cls) {
        //下面这一句无法通过编译
        //List<ILiEventListener<T>>  error= this.eventListenerMap.computeIfAbsent(cls, c -> new ArrayList<>());
        Object iLiEventListeners = this.eventListenerMap.computeIfAbsent(cls, c -> new ArrayList<>());
        return (List<ILiEventListener<T>>) iLiEventListeners;
    }
```

## 获取泛型的类型
可以通过增加一个泛型方法来实现。

```java
public interface ILiEventListener<T> {

    void listen(T event);

    Class<T> listenType();
}

```
## `void`类型的范型方法

```java
private <T> void set(T t) {
  System.out.println(t);
}
```

## 泛型在map中的一些使用

```java
//可以保证存储在cache中的key和value一定是同一个泛型的
Map<Class<?>,Object> cache = new HashMap<>();

public <T>void put(Class<T> type,T value){
    cache.put(type,value);
}

public <T>T get(Class<T> type){
    return cache.get(type);
}

```

## 创建泛型数组


```java

//java.lang.reflect.Array.newInstance(Class<T> componentType, int length)

ArrayWithTypeToken<Integer> arrayToken = new ArrayWithTypeToken<Integer>(Integer.class, 100);
Integer[] array = arrayToken.create();

```

## 获取类声明的泛型


当继承一个泛型类时指定了具体的泛型，通过反射获取该泛型的具体类型

```java
ParameterizedType  paramterizedType =(ParameterizedType) this.getClass().getGenericSuperclass();
paramterizedType.getActualTypeArguments()
// 对于 public class EditRelateJavaAction extends SelectionContextMenuAction<FlowNode> 来说，其值就为FlowNode


```

## `java`运行时无法捕获`ClassCastException`的解决办法

```java
 private static <T> T get(Object o, T def) {
   try {
     return (T) def.getClass().cast(o);
   } catch (Throwable e) {
     return (T) def;
   }
 }
```

通过查看字节码就可以了解,直接 `return (T) value` 是在方法外检测`cast`

## 数组的 class

```java
Object[].class
```

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


## 数组对象与数组的区别

![genericsAndReflect_代码示例.png](genericsAndReflect_代码示例.png)

其字节码如下

```java
public class com/leaderli/demo/TheClassTest {

  public <init>()V
   L0
    LINENUMBER 3 L0
    ALOAD 0
    INVOKESPECIAL java/lang/Object.<init> ()V
    RETURN
   L1
    LOCALVARIABLE this Lcom/leaderli/demo/TheClassTest; L0 L1 0
    MAXSTACK = 1
    MAXLOCALS = 1

  public static varargs test([Ljava/lang/Object;)V
   L0
    LINENUMBER 8 L0
    RETURN
   L1
    LOCALVARIABLE args [Ljava/lang/Object; L0 L1 0
    MAXSTACK = 0
    MAXLOCALS = 1

  public static main([Ljava/lang/String;)V
   L0
    LINENUMBER 10 L0
    ICONST_0                     //压入0
    ANEWARRAY java/lang/String   //弹出栈顶，生成一个栈顶数值长度的String数组
    ASTORE 1                     //弹出栈顶到本地变量1，即p1
   L1
    LINENUMBER 11 L1
    ICONST_1                     //压入1
    ANEWARRAY java/lang/Object   //生成一个长度为1的Object数组
    DUP                          //复制一份引用至栈顶
    ICONST_0                     //压入0
    ALOAD 1                      //压入本地变量1，即p1
    AASTORE                      //弹出栈顶的引用型数值（value）、数组下标（index）、数组引用（arrayref）出栈，将数值存入对应的数组元素中。这里的意思是将p1存入到方法test的形参数组角标0的位置
    INVOKESTATIC com/leaderli/demo/TheClassTest.test ([Ljava/lang/Object;)V // 弹出栈顶所有元素作为参数调用方法，方法返回值会被压入栈顶，因方法返回类型为V，操作栈则清空
   L2
    LINENUMBER 12 L2
    ALOAD 1                       //压入本地变量1
    CHECKCAST [Ljava/lang/Object; //类型检查
    CHECKCAST [Ljava/lang/Object; //类型检查
    ASTORE 2                      //弹出栈顶到本地变量2，即p2
   L3
    LINENUMBER 13 L3
    ALOAD 2                       //压入本地变量1，即p1
    INVOKESTATIC com/leaderli/demo/TheClassTest.test ([Ljava/lang/Object;)V // 弹出栈顶所有元素作为参数调用方法，方法返回值会被压入栈顶，因方法返回类型为V，操作栈则清空
   L4
    LINENUMBER 14 L4
    RETURN
   L5
    LOCALVARIABLE args [Ljava/lang/String; L0 L5 0
    LOCALVARIABLE p1 Ljava/lang/Object; L1 L5 1  //描述符L表示它是对象类型
    LOCALVARIABLE p2 [Ljava/lang/Object; L3 L5 2 //描述符[L表示它是对象数组类型
    MAXSTACK = 4
    MAXLOCALS = 3
}

```


## 桥接子类获取泛型

父类泛型可以使用

```java
ParameterizedTypeImpl type = (ParameterizedTypeImpl) MyNode.class.getGenericSuperclass();
//具体每个泛型
type.getActualTypeArguments()
```

接口泛型

```java
ParameterizedTypeImpl[] types = (ParameterizedTypeImpl[]) MyNode.class.getGenericInterfaces();

```

## Spring 注入桥接子类注意

```java
public interface Generic<T,R> {}
@Component
public class G1 implements Generic<Object, Collection> {}
public class G2 implements Generic<Object, List> {}
public class G3<T> implements Generic<T, List> {}
public class G4 implements Generic<String, List> {}
```

```java

   @Autowired
   List<Generic> generics; //G1 G2 G3 G4
   @Autowired
   List<Generic<?, ? extends Collection>> generics; //G1 G2 G3 G4
   @Autowired
   List<Generic<?, Collection>> generics;//G1
   @Autowired
   List<Generic<Object, ? extends Collection>> generics; //G1 G2 G3
   @Autowired
   List<Generic<Object, Collection>> generics; //G1 G2
```
