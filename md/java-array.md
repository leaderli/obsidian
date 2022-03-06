---
title: java数组
date: 2019-11-12 09:36:54
categories: java
tags:
  - jvm
---


## 数组对象与数组的区别

![genericsAndReflect_代码示例.png](genericsAndReflect_代码示例.png)

其字节码如下

```java
public class com/leaderli/demo/TheClassTest {

  public <init>()V
   L0
    LINENUMBER 3 L0
    ALOAD 0
    INVOKESPECIAL java/lang/Object.<init> ()V
    RETURN
   L1
    LOCALVARIABLE this Lcom/leaderli/demo/TheClassTest; L0 L1 0
    MAXSTACK = 1
    MAXLOCALS = 1

  public static varargs test([Ljava/lang/Object;)V
   L0
    LINENUMBER 8 L0
    RETURN
   L1
    LOCALVARIABLE args [Ljava/lang/Object; L0 L1 0
    MAXSTACK = 0
    MAXLOCALS = 1

  public static main([Ljava/lang/String;)V
   L0
    LINENUMBER 10 L0
    ICONST_0                     //压入0
    ANEWARRAY java/lang/String   //弹出栈顶，生成一个栈顶数值长度的String数组
    ASTORE 1                     //弹出栈顶到本地变量1，即p1
   L1
    LINENUMBER 11 L1
    ICONST_1                     //压入1
    ANEWARRAY java/lang/Object   //生成一个长度为1的Object数组
    DUP                          //复制一份引用至栈顶
    ICONST_0                     //压入0
    ALOAD 1                      //压入本地变量1，即p1
    AASTORE                      //弹出栈顶的引用型数值（value）、数组下标（index）、数组引用（arrayref）出栈，将数值存入对应的数组元素中。这里的意思是将p1存入到方法test的形参数组角标0的位置
    INVOKESTATIC com/leaderli/demo/TheClassTest.test ([Ljava/lang/Object;)V // 弹出栈顶所有元素作为参数调用方法，方法返回值会被压入栈顶，因方法返回类型为V，操作栈则清空
   L2
    LINENUMBER 12 L2
    ALOAD 1                       //压入本地变量1
    CHECKCAST [Ljava/lang/Object; //类型检查
    CHECKCAST [Ljava/lang/Object; //类型检查
    ASTORE 2                      //弹出栈顶到本地变量2，即p2
   L3
    LINENUMBER 13 L3
    ALOAD 2                       //压入本地变量1，即p1
    INVOKESTATIC com/leaderli/demo/TheClassTest.test ([Ljava/lang/Object;)V // 弹出栈顶所有元素作为参数调用方法，方法返回值会被压入栈顶，因方法返回类型为V，操作栈则清空
   L4
    LINENUMBER 14 L4
    RETURN
   L5
    LOCALVARIABLE args [Ljava/lang/String; L0 L5 0
    LOCALVARIABLE p1 Ljava/lang/Object; L1 L5 1  //描述符L表示它是对象类型
    LOCALVARIABLE p2 [Ljava/lang/Object; L3 L5 2 //描述符[L表示它是对象数组类型
    MAXSTACK = 4
    MAXLOCALS = 3
}

```
