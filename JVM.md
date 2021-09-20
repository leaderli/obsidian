
## 类文件的数据结构
```c++
ClassFile {
    u4             magic;
    u2             minor_version;
    u2             major_version;
    u2             constant_pool_count;
    cp_info        constant_pool[constant_pool_count-1];
    u2             access_flags;
    u2             this_class;
    u2             super_class;
    u2             interfaces_count;
    u2             interfaces[interfaces_count];
    u2             fields_count;
    field_info     fields[fields_count];
    u2             methods_count;
    method_info    methods[methods_count];
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

## 常量池

JVM 会为每个类型维护一个常量池，运行时的数据结构有点类似一个符号表，尽管它包含的信息更多。java 的字节码操作需要对应的数据，但是这些数据太大了，存储在字节码里不合适，它们都被存储在常量池里，而字节码包含一个常量池的引用，当类文件生成的时候，其中一块就是常量池