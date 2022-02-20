
### 概述
[📒 JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/#overview)

一个用于生成笛卡尔积参数的框架

[📒 junit-pioneer](https://github.com/junit-pioneer/junit-pioneer)
### 快速入门

```xml
<dependency>
	<groupId>org.junit.jupiter</groupId>
	<artifactId>junit-jupiter-engine</artifactId>
	<version>5.8.1</version>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>org.junit.jupiter</groupId>
	<artifactId>junit-jupiter-api</artifactId>
	<version>5.8.1</version>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>org.junit.jupiter</groupId>
	<artifactId>junit-jupiter-params</artifactId>
	<version>5.8.1</version>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>org.junit.platform</groupId>
	<artifactId>junit-platform-commons</artifactId>
	<version>1.8.1</version>
	<scope>test</scope>
</dependency>

<dependency>
	<groupId>org.junit-pioneer</groupId>
	<artifactId>junit-pioneer</artifactId>
	<version>1.5.0</version>
	<scope>test</scope>
</dependency>
```

```java
public class TestJunit {

    @ParameterizedTest
    @ValueSource(strings = { "racecar", "radar", "able was I ere I saw elba" })
    void parameters(String candidate) {
        System.out.println(candidate);
    }
}

```

![[Pasted image 20220221002914.png]]



```java
@CartesianTest  
void myCartesianTestMethod(  
	@CartesianTest.Values(ints = { 1, 2 }) int x ,  
	@CartesianTest.Values(ints = { 3, 4 }) int y ) {  
  
 System.out.println(x+" "+ y);  
}
```

![[Pasted image 20220221003545.png]]
