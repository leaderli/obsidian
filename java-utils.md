---
title: java-工具类
date: 2020-06-04 22:31:25
categories: java
tags:
  - tips
---

### EventBus

`maven`

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>18.0</version>
</dependency>
```

`post`事件，注册的 listener 中注解了`@Subscribe`的方法会被执行，该方法的参数的类型需要与`event`类型一致，若没有类型一致的`@Subscribe`，则由参数类型`DeadEvent`的`listener`统一处理

```java
public class EventBusTest {
    @Test
    public void test() {
        EventBus eventBus = new EventBus();
        EventListener eventListener = new EventListener();
        eventBus.register(eventListener);
        eventBus.post("hello");
        eventBus.post(123);
    }
    public static class EventListener {
        @Subscribe
        public void stringEvent(String event) {
            System.out.println("event = " + event);
        }
        @Subscribe
        public void handleDeadEvent(DeadEvent deadEvent) {
            System.out.println("deadEvent = " + deadEvent);
        }
    }
}

```

### `junit` 断言异常

```java
public class Student {
    public boolean canVote(int age) {
        if (i<=0) throw new IllegalArgumentException("age should be +ve");
        if (i<18) return false;
        else return true;
    }
}
public class TestStudent{

    @Rule
    public ExpectedException thrown= ExpectedException.none();

    @Test
    public void canVote_throws_IllegalArgumentException_for_zero_age() {
        Student student = new Student();
        thrown.expect(IllegalArgumentException.class);
        thrown.expectMessage("age should be +ve");
        student.canVote(0);
    }
}
```

### `poi`

在生成`excel`时，当为单元格填充内容为数字时，生成的 excel 的数字单元格左上角提示绿色小三角。可在填充单元格值时使用`Double`类型

```java
XSSFCell cell = row.createCell(cellNum);
cell.setCellType(Cell.CELL_TYPE_STRING);
if(value.matches("\\d+")){
    cell.setCellValue(Double.valueOf(value));
}else{
    cell.setCellValue(value);
}
```

### des 

使用秘钥加密解密

```java
import javax.crypto.Cipher;
import javax.crypto.SecretKey;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.DESKeySpec;

public class DesUtil {


    private static SecretKey getSecretKey(String key) throws Exception {
        DESKeySpec dks = new DESKeySpec(key.getBytes());
        SecretKeyFactory skf = SecretKeyFactory.getInstance("DES");
        return skf.generateSecret(dks);
    }

    public static String encrypt(String input, String key) throws Exception {
        Cipher ecipher = Cipher.getInstance("DES");
        ecipher.init(Cipher.ENCRYPT_MODE, getSecretKey(key));
        byte[] utf8 = input.getBytes(Base64Utils.DEFAULT_CHARSET);
        byte[] enc = ecipher.doFinal(utf8);
        return Base64Utils.encodeToString(enc);

    }

    public static String decrypt(String input, String key) throws Exception {
        Cipher dcipher = Cipher.getInstance("DES");
        dcipher.init(Cipher.DECRYPT_MODE, getSecretKey(key));
        byte[] dec = Base64Utils.decodeFromString(input);
        byte[] utf8 = dcipher.doFinal(dec);
        return new String(utf8, Base64Utils.DEFAULT_CHARSET);

    }

}
```

`Base64`的工具类，参考spring的`Base64Utils`

```java
import java.nio.charset.Charset;
import java.nio.charset.StandardCharsets;
import java.util.Base64;

public class Base64Utils {

    public static final Charset DEFAULT_CHARSET = StandardCharsets.UTF_8;

    public static byte[] encode(byte[] src) {
        if (src.length == 0) {
            return src;
        }
        return Base64.getEncoder().encode(src);
    }

    public static byte[] decode(byte[] src) {
        if (src.length == 0) {
            return src;
        }
        return Base64.getDecoder().decode(src);
    }

    public static String encodeToString(byte[] src) {
        if (src.length == 0) {
            return "";
        }
        return new String(encode(src), DEFAULT_CHARSET);
    }

    public static byte[] decodeFromString(String src) {
        if (src.isEmpty()) {
            return new byte[0];
        }
        return decode(src.getBytes(DEFAULT_CHARSET));
    }
}
```


###  AssertJ
```xml
<dependency>
  <groupId>org.assertj</groupId>
  <artifactId>assertj-core</artifactId>
  <!-- use 2.5.0 for Java 7 projects -->
  <version>3.5.2</version>
  <scope>test</scope>
</dependency>
```