
### 概述
一个测试java方法性能的框架，全称Java Moicrobenchmark Harness(微基准测试框架)
[📒github](https://github.com/openjdk/jmh)
[参考](https://mp.weixin.qq.com/s/wOUe4_SRwxidkpBAcG8L8g)

java的基准测试需要注意的几个点
- 测试前需要预热，JVM默认的执行模式是JIT编译与解释混合执行。JVM通过热点代码统计分析，识别高频方法的调用、循环体、公告模块等，基于JIT动态编译技术，会将热点代码转换为机器码，直接交给CPU执行。
- 防止无用代码进入测试方法中
- 并发测试
- 测试结构展示


### 快速入门

```xml
<dependency>
	<groupId>org.openjdk.jmh</groupId>
	<artifactId>jmh-core</artifactId>
	<version>1.23</version>
</dependency>
<dependency>
	<groupId>org.openjdk.jmh</groupId>
	<artifactId>jmh-generator-annprocess</artifactId>
	<version>1.23</version>
</dependency>
```

```java
package com.leaderli.demo;

import org.openjdk.jmh.annotations.*;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.util.LinkedList;
import java.util.List;
import java.util.concurrent.TimeUnit;

/**
 * @author leaderli
 * @since 2022/2/20
 */
@State(Scope.Benchmark)
@OutputTimeUnit(TimeUnit.SECONDS)
@Threads(Threads.MAX)
public class TestJMH {


    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(TestJMH.class.getSimpleName())
                .forks(1)
                .warmupIterations(2)
                .measurementIterations(2)
                .build();

        new Runner(opt).run();
    }
    private static final int SIZE = 10000;

    private List<String> list = new LinkedList<>();

    @Setup
    public void setUp() {
        for (int i = 0; i < SIZE; i++) {
            list.add(String.valueOf(i));
        }
    }

    @Benchmark
    @BenchmarkMode(Mode.Throughput)
    public void forIndexIterate() {
        for (int i = 0; i < list.size(); i++) {
            list.get(i);
            System.out.print("");
        }
    }

    @Benchmark
    @BenchmarkMode(Mode.Throughput)
    public void forEachIterate() {
        for (String s : list) {
            System.out.print("");
        }
    }
}

```

报告的结果如下

>Benchmark                 Mode  Cnt    Score   Error  Units
>TestJMH.forEachIterate   thrpt    2  600.509          ops/s
>TestJMH.forIndexIterate  thrpt    2  189.286          ops/s



### 注解介绍

#### @BenchmarkMode

|类型            |描述              |
|--------------|----------------|
|Throughput    |每段时间执行的次数，一般是秒  |
|AverageTime   |平均时间，每次操作的平均耗时  |
|SampleTime    |在测试中，随机进行采样执行的时间|
|SingleShotTime|在每次执行中计算耗时      |
|All           |所有模式            |


#### @Benchmark

方法级注解，表示该方法是需要进行benchmark的对象，用法和Junit的`@Test`类似




