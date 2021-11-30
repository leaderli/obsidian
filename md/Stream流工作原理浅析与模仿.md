---
title: Stream流工作原理浅析与模仿
date: 2019-11-17 23:53:07
categories: java
tags:
  - 源码
  - 轮子
---

参考[Stream 流水线原理](https://www.jianshu.com/p/893fb6febc70)，我们尝试模仿写一个简单的，能说明其基本工作原理的。
我们将 Stream 的执行过程定义为一个个操作节点，那么一般情况下，Stream 的执行过程主要分为以下部分

1. 源节点
2. 中间操作节点
3. 终止操作节点

通常我们会将多个操作构建成作用链， 将每个元素依次让作用链进行处理 ，终止操作收集作用链处理的结果

我们需要实现以下几个特征

1. 要实现链式调用，那么需要定义一个 Stream 的接口来声明所有操作的方法，并且接口的方法的返回值也是 Stream
2. 要实现作用链，那么需要定义一个链表，将操作串联起来。在 Stream 中的链式调用方法中，需要将当前操作添加到这个链表尾部。
3. 仅当调用终止操作节点时，才可能开始对每个元素依次执行作用链，那么我们首先需要找到链表的头
4. 一般情况下，数据是基于某种泛型的，所以操作链表肯定也是基于泛型的，但是有些函数类似`mapper`，操作前后的泛型可能会发生变换，这就要求这个链表出入参的泛型可不同


## 使用`AND`,`OR`,`NOT`拆分复杂`boolean`操作

参照`Stream`的工作方式，我们将运算逻辑缓存，仅在终结节点`end()`才开始计算
要实现链式调用，那么方法`and()`,`or()`,`not()`,`test()`就需要返回类型相同的对象，我们定义`PipeLine`类来进行链式调用
对于我们需要实现的`boolean`工具，我们需要关心的问题

1. 如何进行计算
2. 何时结束
3. 规则校验
4. 运算结果如何保存

我们定义`Sink`类来保存运算逻辑:

1. `accpet`方法，进行逻辑运算以及流程走向
2. `cancel`方法，决定是否需要终结流程
3. `valid`方法，校验链式调用语法是否合法，因为连续调用两次`and`，是无法进行计算的
4. 通过外部传递的`Bool`对象来保存运算结果

```java
package com.leaderli.demo.bool;


import java.util.Arrays;
import java.util.function.Predicate;

public class PipeLine<T> {


    /**
     * 在调用end时，需要找到第一个节点进行运算
     */
    private PipeLine<T> prev;
    /**
     * 存储结果，仅在显式new PipeLine时初始化
     */
    private Bool bool;
    /**
     * 每个PipeLine绑定一个操作
     */
    private Sink<T> sink;


    private PipeLine(Bool bool) {
        this.bool = bool;
    }

    public PipeLine() {
        this.bool = new Bool();
        begin();
    }

    /**
     * 本身不做任何逻辑运算，仅锚定开始位置
     */
    private void begin() {
        this.sink = new Sink<T>(bool, Sink.Type.BEGIN) {
            @Override
            public Sink<T> accept(T test) {
                return next;
            }

            @Override
            public void valid() {
                valid(Type.TEST, Type.NOT);

            }
        };
        this.prev = null;
    }

    /**
     * 实现链表
     *
     * @param sink 当前运算逻辑
     * @param type 运算结果
     * @return 新增链表节点并返回
     */
    private PipeLine<T> add(Sink<T> sink, Sink.Type type) {
        PipeLine<T> pipeLine = new PipeLine<>(this.bool);
        pipeLine.prev = this;
        pipeLine.sink = sink;
        return pipeLine;
    }

    public PipeLine<T> test(Predicate<T> predicate) {
        assert predicate != null;
        Sink<T> sink = new Sink<T>(bool, Sink.Type.TEST) {
            @Override
            public void valid() {
                valid(Type.END, Type.AND, Type.OR);
            }
        };
        sink.predicate = predicate;
        return add(sink, Sink.Type.TEST);
    }

    /**
     * 对于or来说，如果上一个运算逻辑为true，则整个表达式都为true，所以可以直接结束
     * 否则可以直接忽略or操作前面的运算结果
     */
    public PipeLine<T> or() {
        Sink<T> sink = new Sink<T>(bool, Sink.Type.OR) {
            @Override
            public boolean cancel(Bool bool) {
                return bool.result;
            }

            @Override
            public Sink<T> accept(T test) {
                return this.next;
            }

            @Override
            public void valid() {
                valid(Type.TEST, Type.NOT);
            }
        };
        return add(sink, Sink.Type.OR);
    }

    /**
     * 返回下一个test的否定
     */
    public PipeLine<T> not() {
        Sink<T> sink = new Sink<T>(bool, Sink.Type.NOT) {

            @Override
            public Sink<T> accept(T test) {
                Sink<T> sink = this.next;
                this.next = new Sink<T>(bool, Type.TEST) {
                    @Override
                    public Sink<T> accept(T test) {
                        Sink<T> accept = sink.accept(test);
                        this.bool.result = !this.bool.result;
                        return accept;
                    }
                };
                return this.next;
            }

            @Override
            public void valid() {
                valid(Type.TEST);
            }
        };
        return add(sink, Sink.Type.NOT);
    }

    /**
     * 对于or来说，如果上一个运算逻辑为false，则整个表达式都为false，所以可以直接结束
     * 否则可以直接忽略or操作前面的运算结果
     */
    public PipeLine<T> and() {
        Sink<T> sink = new Sink<T>(bool, Sink.Type.AND) {

            @Override
            public boolean cancel(Bool bool) {
                return !bool.result;
            }

            @Override
            public Sink<T> accept(T test) {
                return this.next;
            }

            @Override
            public void valid() {
                valid(Type.TEST, Type.NOT);


            }
        };
        return add(sink, Sink.Type.OR);
    }

    /**
     * 向前查找PipeLine，同时将Sink链接起来
     *
     * @return 返回链表第一个节点，即BEGIN
     */
    public PipeLine<T> end() {
        PipeLine<T> pipeLine = this;
        Sink<T> temp = pipeLine.sink;
        //最后一个操作执行一个一定会cancel的终结节点
        temp.next = new Sink<T>(bool, Sink.Type.END) {
            @Override
            public boolean cancel(Bool result) {
                return true;
            }
        };
        PipeLine<T> pr = pipeLine.prev;
        while (pr != null) {
            pipeLine = pr;
            pr = pipeLine.prev;
            pipeLine.sink.next = temp;
            temp = pipeLine.sink;
        }
        pr = null;
        return pipeLine;
    }

    /**
     * 依次执行Sink，直到触发cancel或者所有Sink执行完成,支持多次操作
     *
     * @param test 数据
     * @return 逻辑运算结果
     */
    public boolean accept(T test) {
        return forSink(sink, test);
    }

    private static class Bool {
        boolean result = false;
    }

    private static class Sink<T> {
        /**
         * 标记操作的类型，主要用来校验表达式是否合法
         */
        public enum Type {
            BEGIN,
            TEST,
            NOT,
            OR,
            END,
            AND
        }

        protected Type type;

        public Sink(Bool bool, Type type) {
            this.bool = bool;
            this.type = type;
        }

        Bool bool;
        Predicate<T> predicate;
        /**
         * 使用链表的方式，将所有操作步骤串联起来
         */
        Sink<T> next;

        /**
         * 是否需要提前结束表达式
         */
        public boolean cancel(Bool result) {
            return false;
        }

        /**
         * @param test 断言
         * @return 返回下一个操作
         */
        public Sink<T> accept(T test) {
            this.bool.result = this.predicate.test(test);
            return next;
        }

        /**
         * 表达式是否合法，一般只需要考虑当前类型操作的下一个操作类型可以为
         */
        public void valid() {
        }

        void valid(Type... types) {
            if (next == null) {
                throw new IllegalStateException("must have end()");
            }
            if (Arrays.stream(types).noneMatch(type -> next.type == type)) {
                throw new IllegalStateException(type + " --> " + Arrays.toString(types) + "; actual is : " + next.type);
            }
        }

    }

    private boolean forSink(Sink<T> sink, T test) {
        while (sink != null) {
            sink.valid();
            if (sink.cancel(bool)) {
                break;
            }
            sink = sink.accept(test);
        }
        return bool.result;
    }

}
```

参考[Stream 流水线原理](https://www.jianshu.com/p/893fb6febc70)



## 优化

通过定义不同的接口来约束行为，类似于[[静态内部类的泛型与建造者模式|建造者模式]]的方式，而不是在运行时去检测。


```java
public interface LiFunction<T, R> {

    R apply(T request, R last);
}
```

定义约束行为的接口

```java
public interface LinterPredicate<T> extends Function<T,Boolean> {  
    LinterCombineOperation<T> and();  
    LinterCombineOperation<T> or();  
}
public interface LinterOperation<T> {  
    LinterPredicate<T> test(Predicate<T> predicate);  
}
public interface LinterNotOperation<T> {  
    LinterOperation<T> not();  
}

//因为not后面仅运行断言类的操作，所以需要和连接符区分开
public interface LinterCombineOperation<T> extends LinterOperation<T>,LinterNotOperation<T> {  
  
}

//定义一个抽象实现类组合接口，可定义多个实现类
public interface LinterLogicPipeLine<T> extends LinterCombineOperation<T>, LinterPredicate<T>{  
}
```

一个双向链表，用于实现链式调用

```java
package com.leaderli.liutil.stream;

public abstract class LiSink<T, R> implements LiFunction<T, R> {

    public final LiSink<T, R> prev;
    public LiSink<T, R> next;


    public LiSink(LiSink<T, R> prev) {
        this.prev = prev;
        if (prev != null) {
            prev.next = this;
        }

    }


    public final R next(T request,R lastValue){
        if(this.next!=null){
            return this.next.apply(request, lastValue);
        }
        return lastValue;
    }


	//找到链表的头部执行断言
    public final R request(T request) {

        LiSink<T, R> prev = this;
        while (prev.prev != null) {
            prev = prev.prev;
        }
        return prev.apply(request, null);
    }

}
```

定义一个基础的断言类，方便用于其他的断言方法，例如`len`,`regex`等

```java
package com.leaderli.liutil.stream;  
  
import java.util.function.Predicate;  
  
public class LiPredicateSink<T> extends LiSink<T, Boolean> {  
  
    public static final boolean NO_NOT_OPERATION = false;  
    public static final boolean IS_NOT_OPERATION = true;  
    private final Predicate<T> predicate;  
  
    public LiPredicateSink(LiSink<T, java.lang.Boolean> prev, Predicate<T> predicate) {  
        super(prev);  
        this.predicate = predicate;  
    }  
  
  
    /**  
 * @param notOperation 标记是否前面为not操作符，如果是not，则predicate的值取反  
 */  
 @Override  
 public Boolean apply(T request, Boolean notOperation) {  
        boolean apply = predicate.test(request);  
        apply = notOperation != apply;  
        return next(request, apply);  
    }  
}
```



具体实现类

```java
package com.leaderli.liutil.stream;

import java.util.function.Predicate;

public class LiLogicPipeLine<T> implements LinterLogicPipeLine<T>{


    private static class Head<T> extends LiSink<T, Boolean> {

        public Head() {
            super(null);
        }

        @Override
        public Boolean apply(T request, Boolean last) {
            return next(request, LiPredicateSink.NO_NOT_OPERATION);
        }
    }

    protected LiSink<T, Boolean> liSink = new Head<>();

    protected LiLogicPipeLine() {

    }


    public static <T> LinterCombineOperation<T> instance() {

        return new LiLogicPipeLine<>();
    }

    @Override
    public Boolean apply(T t) {
        return liSink.request(t);
    }

    @Override
    public LinterOperation<T> not() {
        this.liSink = new LiSink<T, Boolean>(this.liSink) {
            @Override
            public Boolean apply(T request, Boolean last) {
                return next(request, LiPredicateSink.IS_NOT_OPERATION);
            }
        };
        return this;
    }

    @Override
    public LinterPredicate<T> test(Predicate<T> predicate) {

        this.liSink = new LiPredicateSink<>(this.liSink, predicate);

        return this;
    }


    @Override
    public LinterCombineOperation<T> and() {

        this.liSink = new LiSink<T, Boolean>(this.liSink) {
            @Override
            public Boolean apply(T request, Boolean lastPredicateResult) {
                //短路
                if (lastPredicateResult) {
                    return next(request, LiPredicateSink.NO_NOT_OPERATION);
                }
                return false;
            }
        };
        return this;
    }

    @Override
    public LinterCombineOperation<T> or() {
        this.liSink = new LiSink<T, Boolean>(this.liSink) {
            @Override
            public Boolean apply(T request, Boolean lastPredicateResult) {
                //短路
                if (lastPredicateResult) {
                    return true;
                }
                return next(request, LiPredicateSink.NO_NOT_OPERATION);
            }
        };
        return this;
    }

}
```

实际使用

```java
LiLogicPipeLine.instance().not().test(str->false).and().not().test(str->false);  
assert logic.apply("hello");
```

当我们根据具体泛型去扩展行为时，我们可以重新定义接口规范

```java
private interface MyLinterPredicate extends LinterPredicate<String> {  
	
    MyLinterCombineOperation and();  
  
    MyLinterCombineOperation or();  
  
}  
  
private interface MyLinterOperation extends LinterOperation<String> {  
	
    MyLinterPredicate len(int size);  
	
    MyLinterPredicate test(Predicate<String> predicate);  
}  
  
private interface MyLinterNotOperation extends LinterNotOperation<String> {  
	
    MyLinterOperation not();  
}  
  
private interface MyLinterCombineOperation extends MyLinterOperation, MyLinterNotOperation, LinterCombineOperation<String> {  
}  
  
  
private interface MyLinterLogicPipeLine extends MyLinterCombineOperation, MyLinterPredicate {  
  
}
```

通过适配器的方式将实际操作转发给`LiLogicPipeLine`

```java
 public class MyLiLogicPipeLine implements MyLinterLogicPipeLine {

	private final LiLogicPipeLine<String> proxy = (LiLogicPipeLine<String>) LiLogicPipeLine.<String>instance();

	private MyLiLogicPipeLine() {

	}

	public static <T> MyLiLogicPipeLine instance() {

		return new MyLiLogicPipeLine();
	}

	@Override
	public MyLinterCombineOperation and() {
		proxy.and();
		return this;
	}

	@Override
	public MyLinterCombineOperation or() {
		proxy.or();
		return this;
	}

	@Override
	public MyLinterOperation not() {
		proxy.not();
		return this;
	}

	@Override
	public MyLinterPredicate test(Predicate<String> predicate) {
		proxy.test(predicate);
		return this;
	}

	@Override
	public Boolean apply(String s) {
		return proxy.apply(s);
	}

	@Override
	public MyLinterPredicate len(int size) {
		test(str -> size == str.length());
		return this;
	}
}
```

