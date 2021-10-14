
```java
TypeSpec typeSpec = TypeSpec.classBuilder("Dummy")
  .addSuperinterface(Serializable.class) 
  .superclass(Exception.class) 
  .build();
```