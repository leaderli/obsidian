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

## 被压
略

## FLux & Mono

Flux：	是一个具有 0-N个元素的Publisher
Mono：	是一个具有0-1个元素的Publisher


```java
// 创建一个空的Flux
Flux.empty().subscribe(System.out::println);
// 执行过程会抛异常
Flux.error(new RuntimeException()).subscribe(System.out::println);


// 通过generate创建数据， take表示创建多次
Flux.<String>generate(sink -> {  
            Scanner scanner = new Scanner(System.in);  
                sink.next(scanner.next());  
        })
	    .take(5)  
        .map(String::toUpperCase)  
        .subscribe(System.out::println);
```