#java #tips #util #快捷键 


## 设置

![[settings.zip]]


## 调试

IDEA 开发 web 项目时，建议使用`debug`启动模式，这样可以随时打断点调试项目

1. 使用异常类型断点，在抛出异常时快速进入报错点

2. 断点设置`condition`，仅当满足条件时触发断点

3. 断点可设置依赖关系，仅在前置断点触发后再触发

4. `evaluate Expression`可编写代码进行测试

5. 使用`watches`监听属性的变化

6. `variables`直接修改属性值，进行调试

7. 指定线程下触发断点

8. 移动到下个断点

9. option + 左键 查看变量

## 输入

多尝试用自定义模板[官方自定义模板内容函数](https://www.jetbrains.com/help/idea/template-variables.html)

## 快捷键

==⇧⌘F12== 最小化工具窗口

## `gradle`

`gradle`编译特别慢，需要在`idea`设置中设置`HTTP Proxy`

## 跳转源码

`jump to source`快捷键为`⎋(esc)`或`⌘+↓`

## 回到`Project`视图源码处

`select in project view`默认没有快捷键

## 自动分屏到右边

`move right`

## 在分屏中切换

`⌥tab`

## 切换 tab

`switcher` 快捷键为`⌃⇥`

`⇧⌘[`上个 tab
`⇧⌘]`下个 tab

## debug 模式下,开启变量提示

`show values inline`

## 当前行行数高亮

`line number on caret row`

## `live templates`

`ifn`快速判断当前行数变量是否为 null

```java
if (var == null) {

}
```

自定义`live templates`

```java
@org.junit.Test  
public void test$START$() {  
    $END$  
}
```

指定其生效的范围，触发的关键字为`test`，默认`tab`触发补全
![[Pasted image 20210921012204.png]]

## `Hippie completion`

自动输入前面或后面的单词`⌥/` `⌥⇧/`

## `Smart Type`

智能补全，比如说提示使用何种`lambda` `⇧ space`

## 自动补全根据使用频率

`sort suggestions alphabetically`

## `quick Definitions`

弹出窗口快速查看代码`⌥q`

## `quick documentation`

弹出窗口快速查看文档`⌥F1`

## 为报错文件设置提醒色

`File Color`

## 使用`favorite`

## `custome live template`

可以选中代码后，抽取为`template`

## `keymap abbrevation`

添加快捷方式的缩写，方便使用`find action`查找

## `recent location`

最近访问的路径`⌘⇧e`

## `paster form history`

`idea`记录了最近复制过(`⌘c`)的文本

## `adjust code style setting`

选中代码后，使用可以查卡到选中代码所使用的格式选项，可以去调整它，``⌥⏎`

## `breadcrumbs`

使用面包屑导航显示代码类，方法

## `隐藏目录`

`Editor`|`File Types`|`Ignore files and folders`

## `相关问题`

> objc[1474]: Class JavaLaunchHelper is implemented in both /Library/Java/JavaVirtualMachines/jdk1.8.0_144.jdk/Contents/Home/bin/java (0x10b59a4c0) and /Library/Java/JavaVirtualMachines/jdk1.8.0_144.jdk/Contents/Home/jre/lib/libinstrument.dylib (0x10b6764e0). One of the two will be used. Which one is undefined.

`IDEA`菜单`Help`>>`Edit Custom Properties`
在打开的`idea.properties`里加上

```properties
idea.no.launcher=true
```

## 文件打开方式

idea 若使用某种方式打开文件后，`file type`中的编辑器类型下，会有相关联的文件后缀。

## 标记当前段落不格式化

code style|enable formatter markers in comments


##  springboot yml 无自动提示
项目右键点 Modules，然后点 + 号看是否无 spring 选项


##  junit test 一直 build

关闭maven的 delegate IDE build/run action to maven


## junit 输出slf4j日志告警

> SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.

增加maven配置
```xml
<dependency>
	<groupId>org.slf4j</groupId>
	<artifactId>slf4j-api</artifactId>
</dependency>
<dependency>
	<groupId>org.slf4j</groupId>
	<artifactId>slf4j-simple</artifactId>
</dependency>
```


## typescript不检测代码类型

![[Pasted image 20211223185044.png|400]]