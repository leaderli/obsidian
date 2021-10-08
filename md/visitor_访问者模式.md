#todo 

[参考1](https://www.liaoxuefeng.com/wiki/1252599548343744/1281319659110433)
[参考2](https://www.runoob.com/design-pattern/visitor-pattern.html)


访问者模式将 对数据结构（例如文件夹和文件构成的树形结构）的访问，和对其进行的操作（如查找指定后缀文件）分离。数据结构中提供进行访问的接口`accept`，而对数据结构进行操作的`visitor`，由数据结构对象中，以回调的方式在`visitor`中处理操作逻辑。如果需要新增一组操作，那么只需要增加一个新的`visitor`即可。


示例1：

定义访问者接口，定义接口可以操作的数据

```java
public interface Visitor {  
    default void visitorDir(File dir) { }  
    default void visitorFile(File file) { }
}
```

定义能持有文件夹和文件的数据结构

```java
public class FileStruct {
	public final File file;
	public FileStruct(File file) {
		this.file = file;
	}
	public void accept(Visitor visitor) {
		if (file.isDirectory()) {
			visitor.visitorDir(file);
			for (File child : file.listFiles()) {
				new FileStruct(child).accept(visitor);
			}
		} else {
			visitor.visitorFile(file);
		}
	}
}
```

遍历文件夹

```java
FileStruct fileStruct = new FileStruct(new File("D:\\download"));

fileStruct.accept(new Visitor() {
	@Override
	public void visitorDir(File dir) {
		System.out.println("visit dir:" + dir.getName());
	}

	@Override
	public void visitorFile(File file) {
		System.out.println("visit file:" + file.getName());
	}
});
```

当我们想查找exe文件时，我们可以增加一个新的visitor

```java
fileStruct.accept(new Visitor() {  
 @Override  
 public void visitorFile(File file) {  
        if(file.getName().endsWith(".exe")){  
            System.out.println(file.getName());  
        }  
    }  
});
```


示例2
根据数据结构的不同，自动使用多态的方法执行
```java
public interface ComputerPartVisitor {
   public void visit(Computer computer);
   public void visit(Mouse mouse);
   public void visit(Keyboard keyboard);
   public void visit(Monitor monitor);
}
```

```java
public interface ComputerPart {
   public void accept(ComputerPartVisitor computerPartVisitor);
}
```

```java
public class Keyboard  implements ComputerPart {
   @Override
   public void accept(ComputerPartVisitor computerPartVisitor) {
      computerPartVisitor.visit(this);
   }
}
```

```java
public class Monitor  implements ComputerPart {
   @Override
   public void accept(ComputerPartVisitor computerPartVisitor) {
      computerPartVisitor.visit(this);
   }
}
```

```java
public class Mouse  implements ComputerPart {
   @Override
   public void accept(ComputerPartVisitor computerPartVisitor) {
      computerPartVisitor.visit(this);
   }
}
```

```java
public class Computer implements ComputerPart {
	
   ComputerPart[] parts;
	
   public Computer(){
      parts = new ComputerPart[] {new Mouse(), new Keyboard(), new Monitor()};      
   } 
 
   @Override
   public void accept(ComputerPartVisitor computerPartVisitor) {
      for (int i = 0; i < parts.length; i++) {
         parts[i].accept(computerPartVisitor);
      }
      computerPartVisitor.visit(this);
   }
}
```


```java
public class ComputerPartDisplayVisitor implements ComputerPartVisitor {
 
   @Override
   public void visit(Computer computer) {
      System.out.println("Displaying Computer.");
   }
 
   @Override
   public void visit(Mouse mouse) {
      System.out.println("Displaying Mouse.");
   }
 
   @Override
   public void visit(Keyboard keyboard) {
      System.out.println("Displaying Keyboard.");
   }
 
   @Override
   public void visit(Monitor monitor) {
      System.out.println("Displaying Monitor.");
   }
}
```

```java
public class VisitorPatternDemo {
   public static void main(String[] args) {
 
      ComputerPart computer = new Computer();
      computer.accept(new ComputerPartDisplayVisitor());
   }
}
```