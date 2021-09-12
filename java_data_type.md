# java数据类型


## 八个基本类型

| 类型     | 占用字节 |
| :------ | :------- |
| boolean | 1        |
| byte    | 1        |
| char    | 2        |
| short   | 2        |
| int     | 4        |
| float   | 4        |
| long    | 8        |
| double  | 8        |


### 自动装箱与拆箱

基本类型都有对应的包装类型，基本类型与其对应的包装类型之间的赋值使用自动装箱与拆箱完成。自动装箱与拆箱是JVM编译阶段去实现的

```java
public class IntegerTest {


    public static void main(String[] args) {

        int b = test(1);
    }
    public static int test(Integer a ){
        return a;
    }

}
```
通过查看其字节码，我们可以发现，JVM为我们添加了装箱与拆箱的功能。

```java
public static main([Ljava/lang/String;)V
    // parameter  args
   L0
    LINENUMBER 8 L0
    ICONST_1
    INVOKESTATIC java/lang/Integer.valueOf (I)Ljava/lang/Integer;
    INVOKESTATIC com/leaderli/demo/IntegerTest.test (Ljava/lang/Integer;)I
    ISTORE 1
   L1
    LINENUMBER 9 L1
    RETURN
   L2
    LOCALVARIABLE args [Ljava/lang/String; L0 L2 0
    LOCALVARIABLE b I L1 L2 1
    MAXSTACK = 1
    MAXLOCALS = 2

  // access flags 0x9
  public static test(Ljava/lang/Integer;)I
    // parameter  a
   L0
    LINENUMBER 11 L0
    ALOAD 0
    INVOKEVIRTUAL java/lang/Integer.intValue ()I
    IRETURN
   L1
    LOCALVARIABLE a Ljava/lang/Integer; L0 L1 0
    MAXSTACK = 1
    MAXLOCALS = 1
```

我们可以发现装箱使用了方法`Integer.valueOf(I)`，java为其做了一部分的缓存（`-128~127`），我们可通过调整参数`java.lang.Integer.IntegerCache.high`来加大缓存。

```java
static final int low = -128;
static final int high;
static final Integer cache[];
static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
    
```

## 引用类型

引用类型是通过指针去实现的，在64位平台上，占8个字节（若开启指针压缩，则也为4个字节，在32位平台上占4个字节

## String

String 被声明为 final，因此它不可被继承。 内部使用 char 数组存储数据，该数组被声明为 final，这意味着 value 数组初始化之后就不能再引用其它数组。并且 String 内部没有改变 value 数组的方法，因此可以保证 String 不可变。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0
}
```
**不可变的好处** :

1. 可以缓存 hash 值 因为 String 的 hash 值经常被使用，例如 String 用做 HashMap 的 key。不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。 
2.  String Pool 的需要 如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool
3. 线程安全



我们查看一段字节码

```java
String s1 = "123";
String s2 = "123";
```

```java
L0
    LINENUMBER 10 L0
    LDC "123"
    ASTORE 1
L1
    LINENUMBER 11 L1
    LDC "123"
    ASTORE 2
```

> `ldc` 常量池中的常量值（int, float, string reference, object reference）入栈。

我们可以看到String从常量池中引入到JVM中。

```java
public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}
```

String的构造器实际也是引用常量池的String内部的`char[]`数组，当我们尝试修改`char[]`，会影响所有字符串名称一样的String，包括以后新建的变量，通过构造器创建的String不会被放入到字符串常量池中，可通过`intern()`返回其在字符串常量池的引用

```java
String s1 = "123";
String s2 = "123";

Field value = String.class.getDeclaredField("value");
value.setAccessible(true);
value.set(s2,new char[]{'f','u','c','k'});

String s3 = new String("123");
String s4 = "123";

System.out.println(s1);
System.out.println(s2);
System.out.println(s3);
System.out.println(s4);

//fuck
//fuck
//fuck
//fuck



String s0 = "aaa";
String s1 = new String("aaa");
String s2 = new String("aaa");
String s3 = s1.intern();
System.out.println(s1 == s2);           // false
System.out.println(s1.intern() == s3);  // true
System.out.println(s1 == s3);  // false
System.out.println(s0 == s3);  // true
```
