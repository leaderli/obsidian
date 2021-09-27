
### 常见的URL协议

| 协议         | 名称             | 描述    |
| ------------ | ---------------- | --- |
| `http(s)://` | 超文本传输协议   |  通过http(s)访问该资源。 格式 `http(s)://`|
| `ftp://`     | 传输文件协议     |   通过FTP访问资源。格式 `ftp://`  |
| `file://`    | 获取本地文件协议 |     资源是本地计算机上的文件。格式 `file:///`，注意后边应是三个斜杠。|
| `mailto://`  | 发邮件协议       |   资源为电子邮件地址，通过SMTP访问。格式 `mailto:`  |


### url的各个组成部分
字面量参数可选值：

| 参数     | 返回值                               |
| -------- | ------------------------------------ |
| hash     | URL的锚(从"#"开始之后的内容)         |
| protocol | 协议http、https、ftp… 等             |
| hostname | 域名                                 |
| port     | 端口号                               |
| host     | 完整的主机名，包含域名和端口号       |
| origin   | 完整的主机路径，包含协议、域名、端口 |
| pathname | 主机名之后的部分                     |
| search   | 查询字符串                           |
| query    | 查询字符串构成对象                   |
| href     | URL全名                              |


### java中URL

```java
  
URL url = new URL("jar:file:/D:/ProgramFiles/Java/jdk1.8.0_281/jre/lib/charsets.jar!/META-INF");  
System.out.println(url.getProtocol());  
System.out.println(url.getFile());
```
>jar
>file:/D:/ProgramFiles/Java/jdk1.8.0_281/jre/lib/charsets.jar!/META-INF

可直接通过URL打开InputStream 
```java
URL url = new URL("jar:file:/D:/resouce/java/maven/repository/com/leaderli/LiDemo/1.0/LiDemo-1.0.jar!/META-INF/services/com.leaderli.lidemo.ISpiDemo ");  
InputStream inputStream = url.openStream();  
String text = new BufferedReader(new InputStreamReader(inputStream, StandardCharsets.UTF_8)).lines().collect(Collectors.joining("\n"));
```