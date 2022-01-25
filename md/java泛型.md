#java #泛型  #泛型数组

## 泛型 `extends super`

```ad-li
不管是extends或是super，只能使用在变量声明上，实际赋值的时候，一定是指定具体实现类的，这就是extend和super的方法会有所限制
```

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

当我们实际调用这个 mapper 的 apply 时，
- 请求参数为 T 类型，apply方法体内部是可以使用T类型的所有方法的，
- 返回值为 R 类型。

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

java.lang.reflect.Array.newInstance(Class<T> componentType, int length)


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



## 桥接方法

当类定义中的类型参数没有任何限制时，在类型擦除中直接被替换为Object，即形如`<T>`和`<?>`的类型参数都被替换为Object。

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

## 任意泛型
当一个类不包含任何泛型的成员变量时，其可以安全的转换为任意泛型

```java
public class Some<T>{

	public void test(T t){

	}

	@SuppressWarnings("unchecked")  
	public <R> Some<R> toSome() {  
	 return (Some<R>) this;  
	}
}
```