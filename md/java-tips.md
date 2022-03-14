---
title: java tips
date: 2019-08-06 10:50:37
categories: java
tags:
  - java
  - tips
---

### 将常量放在接口中，通过继承该接口，调用常量

```java
public interface ClassConstants{
   int CONSUMER = 1;//接口中变量默认为 static final
   int DISPLAY = 2;
}
```

### 查看 Class 是否是基本类型

```java
clasz.isPrimitive();
```



### 基本类型转换包装类型

```java
Object a = 1;
//此时a自动转换为Integer类型

public <T> void fun(T t){
  System.out.println(t.getClass());
}

//fun方法会自动将1转换为Integer包装类
fun(1);
```

### 查看类是否为基本类型或包装类型

```java
import org.apache.commons.lang3.ClassUtils;
ClassUtils.isPrimitiveOrWrapper(klass)
```

### 读取 properties 中文乱码解决

```java
properties.load(new InputStreamReader(AutoConfig.class.getResourceAsStream("/application.properties"),"utf-8"));
```





### 判断类是否继承自

```java
Father.class.isAssignableFrom(Son.class)
```

### 获取当前执行的方法名,通过方法内的内部类来实现的

```java
new Object(){}.getClass().getEnclosingMethod().getName();
```

### 使用 shell


```ad-info
脚本中如果有使用环境变量的地方，需要先执行source命令
```

```java
//获取当前系统名称，可根据不同系统调用不同的命令
String os = System.getProperty("os.name");
//命令行的参数
String [] para = new String[]{"-l"}
//执行命令
Process ls = Runtime.getRuntime().exec("ls",para, new File("/Users/li/Downloads"));



//获取命令执行结果的inputstream流
String getLs = new BufferedReader(new InputStreamReader(ls.getInputStream())).lines().collect(Collectors.joining(System.lineSeparator()));

//在目录下执行具体的命令
//因为java执行shell无法进行连续性的交互命令，通过封装的bash或者python脚本执行一系列命令是比较好的选择
Process process = Runtime.getRuntime().exec("python test.py", null, new File("/Users/li/Downloads"));
// 部分os需要先输出outputStream流，才能正确取得shell执行结果
Outputstream out = process.getOutputStream();
out.flush();
out.close();

//使用构造器模式
ProcessBuilder builder = new ProcessBuilder();
builder.command("more","test.py");
builder.directory(new File("/Users/li/Downloads"));
//重定向错误流，即System.Err
builder.redirectErrorStream(true);
Process process = builder.start();





// 设定一些环境变量
builder.environment().put("some_variable","uat")
	
// source
builder.command("source","~/.bash_profile")	
```

### 基本类型零值

对基本数据类型来说，对于类变量`static`和全局变量，如果不显式地对其赋值而直接使用，则系统会为其赋予默认的零值，可以根据这个特性，直接用静态类常量来获取基本变量的初始值

```java
public class Primitive {
  public static int i; //默认值0
  public static char c; //默认值'\u0000'
}
```


### `Comparator.comparing`

可是使用 lambda 快速实现`comparable`

```java
Comparator<Player> byRanking
 = (Player player1, Player player2) -> player1.getRanking() - player2.getRanking();
```

类似`Collectors`提供了快捷的`Comparator`方法

```java
Comparator<Player> byRanking = Comparator
  .comparing(Player::getRanking);
Comparator<Player> byAge = Comparator
  .comparing(Player::getAge);
```

### 批量反编译 jar 包

```shell
ls *.jar|xargs -I {} jadx {} -d src
```



### 进制

```java
int x = 0b11;// 二进制
int x = 0B11;// 二进制
int x = 0x11;// 十六进制
int x = 0X11;// 十六进制
int x = 011; //  八进制
```

### 读取文件

```java
String javaSource = new String(Files.readAllBytes(Paths.get("src/Hello.java")));

```

### 返回空的集合

```ad-bug

该集合无法进行add操作
```

```java
return Collections.emptyList();
```

### getResource

```java
// getResource最终调用的是ClassLoader的方法

//返回类加载器加载的当前类的路径
Test.class.getResource(".");
// 返回类加载的根目录
Test.class.getResource("/");
```
### classpath

```java
package foo;

public class Test
{
    public static void main(String[] args)
    {
        ClassLoader loader = Test.class.getClassLoader();
        System.out.println(loader.getResource("foo/Test.class"));
    }
}
//This printed out:
//file:/C:/Users/Jon/Test/foo/Test.class
```

### 扫描所有类

```java
public void test() throws IOException, URISyntaxException {  
  
  
    Enumeration<URL> resources = this.getClass().getClassLoader().getResources("");
  
    while (resources.hasMoreElements()){  
  
        scan(new File(resources.nextElement().toURI()));  
    }  
}  
  
private void scan(File file) {  
    if(file==null||!file.exists()){  
       return;  
    }  
  
    System.out.println(file);  
  
    if(file.isDirectory()){  
        for (File f: Objects.requireNonNull(file.listFiles())) {  
            scan(f);  
        }  
    }  
}
```
### 获取项目中所有jar

可查找classpath目录下所有的[[maven#META-INF|META-INF]]，从而找到所有jar包
```java
  
ClassLoader loader = Thread.currentThread().getContextClassLoader();  
Enumeration<URL> resources = loader.getResources("META-INF");  
Collections.list(resources).stream()  
        .filter(url -> "jar".equals(url.getProtocol()))  
        .map(URL::getFile)  
        .forEach(System.out::println);

```

>file:/D:/resouce/java/maven/repository/junit/junit/4.12/junit-4.12.jar!/META-INF
>file:/D:/resouce/java/maven/repository/org/hamcrest/hamcrest-core/2.1/hamcrest-core-2.1.jar!/META-INF
>file:/D:/ProgramFiles/IDEA2020/lib/idea_rt.jar!/META-INF

根据[stackoverflow](https://stackoverflow.com/questions/25729319/how-does-a-classloader-load-classes-reference-in-the-manifest-classpath)中的回答，[[maven#META-INF|META-INF]]是由JVM底层去加载的




### 文件锁

可以使用文件锁来确保只有一个进程访问该文件，确保其他进程（包括其他虚拟机或者其他操作文件的系统)

```java
RandomAccessFile raFile = new RandomAccessFile(new File(filename), "rw")
FileLock lock = raFile.getChannel().lock();
```

使用 RandomAccessFile 拷贝文件

```java
 public static void appendFile(RandomAccessFile main, RandomAccessFile extra) throws IOException {

        // this method uses NIO direct transfer. It delegates the task
        // to the operating system. Only works under 1.4+
        // Unfortunately, can't use this because it crashes with an out of memory error on big files
        //extra.getChannel().transferTo(0, Long.MAX_VALUE, mainFile.getChannel());

        byte[] buf = new byte[1024 * 1024]; // 1 mb
        main.seek(main.length());
        extra.seek(0);

        while (true) {
            int count = extra.read(buf);
            if (count == -1)
                break;
            main.write(buf, 0, count);
        }
    }
```

### 如何修改 final 修饰符的值

```java
String str = "fuck";

Field value = String.class.getDeclaredField("value");
//Field中包含当前属性的修饰符，通过改变修饰符的final属性，达到重新赋值的功能
Field modifier = Field.class.getDeclaredField("modifiers");
modifier.setAccessible(true);
modifier.set(value, value.getModifiers() & ~Modifier.FINAL);

value.setAccessible(true);
value.set(str, "notfuck".toCharArray());
//修改成功后重新加上final修饰符
modifier.set(value, value.getModifiers() | Modifier.FINAL);}
```


###  HttpServletRequest
HttpServletRequest.getRequestURL()中获取的是请求地址，不是实际的IP地址，当通过nginx转发时，有可能返回nginx地址，想要获取真实IP，使用 [[java-config#3 InetAddress getLocalHost java net UnknownHostException 异常|InetAddress getLocalHost]]


### 动态编译源代码


```java
package com.leaderli.litest;

import org.junit.Test;

import javax.tools.JavaCompiler;
import javax.tools.ToolProvider;
import java.io.File;
import java.io.IOException;
import java.net.URL;
import java.net.URLClassLoader;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;

public class TestCompile {
    @Test
    public void test() throws IOException, ClassNotFoundException, InstantiationException, IllegalAccessException {
        String source = "package test;     " +
                "public class Test implements Runnable{\n" +
                "        @Override\n" +
                "        public void run() {\n" +
                "            System.out.println(\"test compile\");\n" +
                "        }\n" +
                " }";

		//将源码保存到目录下
        File root = new File("/java"); // On Windows running on C:\, this is C:\java.
        File sourceFile = new File(root, "test/Test.java");
        sourceFile.getParentFile().mkdirs();
        Files.write(sourceFile.toPath(), source.getBytes(StandardCharsets.UTF_8));

		// 将源码进行编译到指定根目录下，编译失败会抛出相关异常
        JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
		//0表示编译成功
        int status = compiler.run(null, null, null, sourceFile.getPath());

		//通过定义一个新的类加载器去加载编译后的class
        URLClassLoader classLoader = URLClassLoader.newInstance(new URL[]{root.toURI().toURL()});
		//加载以及实例化类
        Class<?> cls = Class.forName("test.Test", true, classLoader);
        Runnable runnable = (Runnable) cls.newInstance(); 
        runnable.run();
    }
}

```


上述编译代码的方式在tomcat中可能会运行不正常，因为相关的jar可能不在classpath中，可通过如下方式加载。
- [[#获取项目中所有jar]]
-  [[JavaPoet]] `JavaFile.toJavaFileObject()`
```java
public void test(JavaFileObject javaFileObject) throws IOException {

        JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
        JavaFileManager fileManager = compiler.getStandardFileManager(null,null,null);

        List<JavaFileObject> files = new ArrayList<>();
        files.add(javaFileObject);
        List<String> options = new ArrayList<>();

        options.add("-Xlint:unchecked");// 忽略一些检测
        
        options.add("-classpath"); // 将必要的jar添加到classpath
        options.add(LiClassUtil.getAppJars().stream().collect(Collectors.joining(File.pathSeparator)));
        
        options.add("-d"); // 编译后的文件输出目录，
        options.add(this.getClass().getResource("/").getPath());
        
        compiler.getTask(null,fileManager,null,options,null,files).call();

        fileManager.close();

}
```


### 获取所有枚举值

```java
enum Color {  
 RED,GREEN  
}  
  
@Test  
public void test() throws Throwable{  
	for (Color enumConstant : Color.class.getEnumConstants()) {  
		System.out.println(enumConstant);  
	}  
}
```

### 枚举不可以extends但可以implements

```java
interface Mode<E extends Enum<E>> {
	public E combine();
}

enum Color implements Mode<Color> {
        RED, GREEN;

        @Override
        public Color combine() {
            return this;
        }
}
class TextFragment<E extends Enum<E> & Mode<E>> {

	E mode;

	public TextFragment(E mode) {
		this.mode = mode;
	}
}

@Test
public void test() {

	TextFragment<Color> text = new TextFragment<>(Color.GREEN);  
	Assert.assertSame(Color.GREEN, text.mode);
}
```


### 暂停线程的另一种方式

```java
TimeUnit.SECONDS.sleep(1);
```


### 输出java对象的内存占用

```java
System.out.println("int[256]:\t\t"+ObjectSizeCalculator.getObjectSize(new int [256]));  
System.out.println("int[2][128]:\t"+ObjectSizeCalculator.getObjectSize(new int [2][128]));  
System.out.println("int[128][2]:\t"+ObjectSizeCalculator.getObjectSize(new int [128][2]));


int[256]:		1040
int[2][128]:	1080
int[128][2]:	3600
```