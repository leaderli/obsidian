#todo

[官方文档](https://commons.apache.org/proper/commons-configuration/userguide/user_guide.html)


## 依赖

使用到不同的功能时，需要加载不同的依赖

![[Pasted image 20211001025403.png]]

核心为下面三个
```xml
<dependency>  
    <groupId>org.apache.commons</groupId>  
    <artifactId>commons-lang3</artifactId>  
    <version>3.5</version>  
</dependency>
<dependency>  
    <groupId>commons-beanutils</groupId>  
    <artifactId>commons-beanutils</artifactId>  
    <version>1.9.3</version>  
</dependency>  
<dependency>  
    <groupId>org.apache.commons</groupId>  
    <artifactId>commons-configuration2</artifactId>  
    <version>2.7</version>  
</dependency>
```

当使用yaml时，需要依赖
```xml
<dependency>  
    <groupId>org.yaml</groupId>  
    <artifactId>snakeyaml</artifactId>  
    <version>1.29</version>  
</dependency>

```

## 多环境
### yml

读取yml，没有多环境的支持，可以通过如下的方式去实现
```java
private YAMLConfiguration loads(String... paths) {  
    YAMLConfiguration yamlConfiguration = new YAMLConfiguration();  
    Map<String, Object> properties = new HashMap<>();  
  
    for (String path : paths) {  
        try {  
            yamlConfiguration.read(ClassLoader.getSystemResource(path).openStream());  
            yamlConfiguration.getKeys().forEachRemaining(k -> properties.put(k, yamlConfiguration.getProperty(k)));  
  
        } catch (ConfigurationException | IOException e) {  
            e.printStackTrace();  
        }  
    }  
  
    properties.forEach(yamlConfiguration::setProperty);  
  
    return yamlConfiguration;  
  
  
}
```