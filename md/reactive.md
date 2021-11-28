---
tags:
  - reactor
  - stream
  - reactive
---


## Reactive Stream 规范

JVM上的响应流规范
![[Pasted image 20211128211147.png]]

定义了四大组成部分，其实是四个接口
1. Publisher	发布者（生产者）
2. Subscriber	订阅者（消费者）
3. Subscription	订阅
4. Processor	处理者

所谓的响应式编程其实就是有一个发布者出发一个事情，有一个或多个订阅者角色来响应式处理这件事，订阅者需要先订阅发布者的事件，一个发布者可以支持多个订阅者，发布者通过onSubsribe事件通知到订阅者，订阅者通过request(n)方法，告诉发布者自己需要多少请求（称为[[#被压]]或者控流），发布者根据请求通过onNext不停的给订阅者发送事件，直到最后出现onError或onComplete终止事件。各种reactive框架都要遵循这个规范，其中[reactor](https://projectreactor.io/) 就是其中一个实现。

## 使用

```xml
<dependency>  
   <groupId>io.projectreactor</groupId>  
   <artifactId>reactor-core</artifactId>  
   <version>3.0.3.RELEASE</version>  
</dependency>
```
## 背压

backpressure

## FLux & Mono

Flux：	是一个具有 0-N个元素的Publisher
Mono：	是一个具有0-1个元素的Publisher


```java

// 创建一个空的Flux
Flux.empty().subscribe(System.out::println);
// 执行过程会抛异常
Flux.error(new RuntimeException()).subscribe(System.out::println);

Flux.just("1","2")
Mono.just("1")

```

```java
// 通过generate创建数据， take表示创建多次
Flux.<String>generate(sink -> {  
            Scanner scanner = new Scanner(System.in);  
                sink.next(scanner.next());  
        })
	    .take(5)  
        .map(String::toUpperCase)  
        .subscribe(System.out::println);

// 带状态的generate，上次的状态state可以传递给下一次generate
Flux.generate(()->0,(Integer state, SynchronousSink<String> sink)->{  
  
            if(state == 1){  
                sink.next("true");  
                return 0;  
            }  
  
            sink.next("false");  
            return 1;  
        })  
        .take(5)  
        .map(String::toUpperCase)  
        .subscribe(System.out::println);
```


```java
Flux.<String>create(sink->sink.next("123")).subscribe(System.out::println);

```


通过一个Consumer的实现类，将产生事件的发生器暴露出去
```java
private static class SinkHolder<T> implements Consumer<FluxSink<T>> {  
    private FluxSink<T> fluxSink;  
  
    @Override  
 public void accept(FluxSink<T> fluxSink) {  
        this.fluxSink = fluxSink;  
    }  
  
    public void emit(T value) {  
        this.fluxSink.next(value);  
    }  
}  
  
@Test  
public void test4() throws Throwable {  
  
  
    SinkHolder<String> holder = new SinkHolder<>();  
    Flux.<String>create(holder).subscribe(System.out::println);  
  
    CountDownLatch countDownLatch = new CountDownLatch(1);  
  
    new Thread(()->{  
  
  
        holder.emit("hello");  
        holder.emit("reactor");  
  
        try {  
            Thread.sleep(1000);  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
  
        holder.emit("end");  
  
        countDownLatch.countDown();  
  
    }).start();  
    countDownLatch.await();  
  
}
```

由于生产者的开关交了出去，消费者无法预知生产者的生成速度，可以通过背压策略来控制输入，传入 #htext/red ==事件流数量大于request数量==时的处理策略

|        |                      |                                                              |
| ------ | -------------------- | ------------------------------------------------------------ |
| BUFFER | onBackpressureBuffer | 默认策略，多出的事件缓存起来，事情数量超过buffer的长度会报错 |
| DROP   | onBackpressureDrop   | 多余的事件丢弃                                               |
| ERROR  | onBackPressureError  | 报错                                                         |
| LATEST | onBackPressureLatest | 只取最后一个                                                 | 




```java
private static class SlowSubscriber<T> implements Subscriber<T> {

	private final int consumeMills;
	private Subscription subscription;

	private SlowSubscriber(int consumeMills) {
		this.consumeMills = consumeMills;
	}


	@Override
	public void onSubscribe(Subscription subscription) {

		this.subscription = subscription;
		this.subscription.request(1);
	}

	@Override
	public void onNext(T t) {

		try {
			Thread.sleep(consumeMills);
			System.out.println("SlowSubscriber:sleep " + consumeMills + " get " + t);
			this.subscription.request(1);

		} catch (InterruptedException e) {
			e.printStackTrace();
			Thread.currentThread().interrupt();
		}
	}

	@Override
	public void onError(Throwable throwable) {

		throwable.printStackTrace();
	}

	@Override
	public void onComplete() {

	}
}

@Test
public void test5() throws Throwable {
	
	SinkHolder<Integer> holder = new SinkHolder<>();

	Flux.create(holder,
					FluxSink.OverflowStrategy.BUFFER
//                        FluxSink.OverflowStrategy.DROP
//                        FluxSink.OverflowStrategy.ERROR
//                        FluxSink.OverflowStrategy.LATEST
			)
			.onBackpressureBuffer()
//                .onBackpressureDrop(new Consumer<Integer>() {
//                    @Override
//                    public void accept(Integer integer) {
//                        System.out.println("drop "+integer);
//                    }
//                })
//                .onBackpressureError()
//                .onBackpressureLatest()
//                .limitRate(4)
			.publishOn(Schedulers.newSingle("consumer"), 1)
			.subscribe(new SlowSubscriber<>(1000));


	AtomicInteger counter = new AtomicInteger();
	new Thread(() -> {

		while (true) {

			holder.emit(counter.getAndIncrement());
			System.out.println(counter.get());
			try {
				Thread.sleep(200);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			if (counter.get() > 10) {
				break;
			}
		}

	}).start();

	Thread.sleep(10000);

}
```


Flux一些独有创建方法
```java
Flux.fromArray(new String[]{"1", "2"}).subscribe(System.out::println);  

Flux.fromIterable(Arrays.asList("1", "2")).subscribe(System.out::println);  

// stream转换为Flux只能被订阅一次  
Flux.fromStream(Stream.of("1", "2")).subscribe(System.out::println);

Flux.range(0,10).subscribe(System.out::println);
```

Mono一些独有创建方法

```java
Mono.fromCallable(() -> {  
    Thread.sleep(1000);  
    return "123";  
}).subscribe(System.out::println);
```


## subscribe

subscribe() 是一个阻塞方法，调用之前执行链不会真正执行。当订阅者通过subscribe方法让订阅者订阅上自己，有有个onSubscribe事件发生时，订阅者就会通过request方法告知生产者自己需要多少个请求，生产者就会不停调用onNext方法把事件数据传递给订阅者，直到发生onError或onComplete终止。

Subscriber是一个接口

```java
public interface Subscriber<T> {
    void onSubscribe(Subscription var1);

    void onNext(T var1);

    void onError(Throwable var1);

    void onComplete();
}
```

Flux和Mono的subscribe()方法重载了很多个同名方法，例如我们常用的方式是使用一个传入Consumer函数，这个Consumer最终由onNext执行。


`Flux.java`
```java
public final Cancellation subscribe(Consumer<? super T> consumer) {  
   Objects.requireNonNull(consumer, "consumer");  
   return subscribe(consumer, null, null);  
}

public final Cancellation subscribe(Consumer<? super T> consumer,  
      Consumer<? super Throwable> errorConsumer,  
      Runnable completeConsumer,  
      Consumer<? super Subscription> subscriptionConsumer) {  
   LambdaSubscriber<T> consumerAction = new LambdaSubscriber<>(consumer,  
         errorConsumer, completeConsumer, subscriptionConsumer);  
  
   subscribe(consumerAction);  
   return consumerAction;  
}

```

`LambdaSubscriber.java`

```java
@Override  
public final void onNext(T x) {  
   if (x == null) {  
      throw Exceptions.argumentIsNullException();  
   }  
   try {  
      if (consumer != null) {  
         consumer.accept(x);  
      }  
   }  
   catch (Throwable t) {  
      Exceptions.throwIfFatal(t);  
      this.subscription.cancel();  
      onError(t);  
   }  
}
```


或者查看我们上面示例中的`SlowSubscriber`。


## 常用API

1. filter
2. defaultIfEmpty
3. any
4. all
5. reduce
6. collect
7. take
	```java
	Flux.range(0,10).take(5).subscribe(System.out::println);
	// 0
	// 1
	// 2
	// 3
	// 4
	```
8. skip
	```java
	Flux.range(0,10).skip(5).subscribe(System.out::println);
	// 5
	// 6
	// 7
	// 8
	// 9
	```
9. sort
10. count
11. doOnRequest
12. doOnNext
	```java
	Flux.fromArray(new String[]{"1", "2"}).doOnNext(sink->{  
		System.out.println("doOnNext:"+sink);  
	}).subscribe(System.out::println);

	// doOnNext:1
	// 1
	// doOnNext:2
	// 2
	```
13. doOnComplete
14. doOnError
15. doOnSubscribe
16. doOnTerminate
17. doOnCancel
18. zipWith
	```java
	Flux.fromIterable(Arrays.asList("a", "b", "c"))  
			.distinct()  
			.sort()  
			.zipWith(Flux.range(1, Integer.MAX_VALUE), (string, count) -> String.format("%2d. %s", count, string))  
			.subscribe(System.out::println);
	 // 1. a
	 // 2. b
	 // 3. c
	```
19. concatWith
	```java
	Flux.fromIterable(Arrays.asList("a", "b", "c"))  
		    .concatWith(Mono.just("d"))
			.distinct()  
			.sort()  
			.zipWith(Flux.range(1, Integer.MAX_VALUE), (string, count) -> String.format("%2d. %s", count, string))  
			.subscribe(System.out::println);
	 // 1. a
	 // 2. b
	 // 3. c
     // 4. d
	```
	
20. delaySubscriptionMillis

	```java
	// 延迟订阅信息，不是每条消息
	CountDownLatch latch = new CountDownLatch(1);  
	Flux.just("1").delaySubscriptionMillis(2000).subscribe(next -> {  
		System.out.println(next);  
		latch.countDown();  
	});  

	latch.await();
	```
