# ASM

ASM是一个生成java字节码的框架，[官网地址](https://asm.ow2.io/index.html)

```maven
<dependency>
    <groupId>org.ow2.asm</groupId>
    <artifactId>asm</artifactId>
    <version>6.0</version>
</dependency>
<dependency>
    <groupId>org.ow2.asm</groupId>
    <artifactId>asm-util</artifactId>
    <version>6.0</version>
</dependency>
```

一个用于生成`HelloWorld`的demo

```java
package com.leaderli.liutil;

import org.objectweb.asm.ClassWriter;
import org.objectweb.asm.MethodVisitor;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.net.URL;
import java.net.URLClassLoader;
import java.util.Arrays;

import static org.objectweb.asm.Opcodes.*;

public class TestAM {
    public static class ByteClassLoader extends URLClassLoader {

        public ByteClassLoader(URL[] urls, ClassLoader parent) {
            super(urls, parent);
        }

        @Override
        protected Class<?> findClass(final String name) throws ClassNotFoundException {
            byte[] classBytes = new TestAM().serializeToBytes(name);
            if (classBytes != null) {
                return defineClass(name, classBytes, 0, classBytes.length);
            }
            return super.findClass(name);
        }

    }

    public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {
        ClassLoader classLoader = TestAM.class.getClassLoader();
        Class<?> fuck = new ByteClassLoader(new URL[]{}, classLoader).findClass("HelloWorld");

        System.out.println(fuck.getName());
        for (Method method : fuck.getMethods()) {
            if (method.getName().startsWith("main"))
                method.invoke(null);
        }

    }

    ClassWriter cw = new ClassWriter(0);

    public byte[] serializeToBytes(String outputClazzName) {
        cw.visit(V1_8, ACC_PUBLIC + ACC_SUPER, outputClazzName,
                null, "java/lang/Object", null);
        addStandardConstructor();
        for (int i = 0; i < 10; i++) {

            addMainMethod(i);
        }
        cw.visitEnd();
        return cw.toByteArray();
    }

    private void addMainMethod(int id) {
        MethodVisitor mv =
                cw.visitMethod(ACC_PUBLIC + ACC_STATIC,
                        "main" + id, "()V", null, null);
        mv.visitCode();
        mv.visitFieldInsn(GETSTATIC, "java/lang/System",
                "out", "Ljava/io/PrintStream;");
        mv.visitLdcInsn("Hello World! " + id);
        mv.visitMethodInsn(INVOKEVIRTUAL, "java/io/PrintStream",
                "println", "(Ljava/lang/String;)V", false);
        mv.visitInsn(RETURN);
        mv.visitMaxs(3, 3);
        mv.visitEnd();
    }

    private void addStandardConstructor() {
        MethodVisitor mv =
                cw.visitMethod(ACC_PUBLIC, "<init>", "()V", null, null);
        mv.visitVarInsn(ALOAD, 0);
        mv.visitMethodInsn(
                INVOKESPECIAL, "java/lang/Object", "<init>", "()V", false);
        mv.visitInsn(RETURN);
        mv.visitMaxs(1, 1);
        mv.visitEnd();
    }
}

```

## 生成

原始类型的表示方法
- `'V' - void`
- `'Z' - boolean`
- `'C' - char`
- `'B' - byte`
- `'S' - short`
- `'I' - int`
- `'F' - float`
- `'J' - long`
- `'D' - double`

Class类型的表示方法
- `L<class>`;
- `Ljava/io/ObjectOutput`;
- `Ljava/lang/String`;

数组则在类型前加`[`

- ``

例如：
`public void method(): ()V`
`public void method(String s, int i): (Ljava/lang/String;I)V`
`public String method(String s, int i, boolan flag):(Ljava/lang/String;IZ)Ljava/lang/String`