
```java
//使用公共的线程池
ForkJoinPool pool = ForkJoinPool.commonPool();
```
无返回值的调用

```java
class PrintStack extends RecursiveAction{  
  
  
    private static final int THRED_HOLD = 9;  
  
  
  
    private final int start;  
    private final  int end;  
  
    private PrintStack(int start, int end) {  
        this.start = start;  
        this.end = end;  
    }  
  
  
    @Override  
    protected void compute() {  
        if(end-start<THRED_HOLD){  
  
            for (int i = start ; i <end ; i++) {  
  
                System.out.println(Thread.currentThread().getName()+" i="+i);  
            }  
        }else{  
  
            int middle  = (start+end)/2;  
  
            PrintStack firstTask  = new PrintStack(start, middle);  
            PrintStack secondTask= new PrintStack(middle+1, end);  
  
            invokeAll(firstTask,secondTask);  
        }  
  
    }  
}
```

```java
ForkJoinPool pool = new ForkJoinPool();  
  
PrintStack task= new PrintStack(0, 50);  
pool.submit(task);  
  
pool.awaitTermination(2, TimeUnit.SECONDS);  
pool.shutdown()
```



有返回值的调用
```java
class CalculateStack extends RecursiveTask<Integer> {  
  
  
    private static final int THRED_HOLD = 9;  
  
  
  
    private final int start;  
    private final  int end;  
  
    private CalculateStack(int start, int end) {  
        this.start = start;  
        this.end = end;  
    }  
  
  
    @Override  
    protected Integer compute() {  
        if(end-start<THRED_HOLD){  
  
            int result = 0;  
            for (int i = start ; i <end ; i++) {  
  
                result +=i;  
            }  
            return result;  
        }else{  
  
            int middle  = (start+end)/2;  
  
            CalculateStack firstTask  = new CalculateStack(start, middle);  
            CalculateStack secondTask= new CalculateStack(middle+1, end);  
  
            invokeAll(firstTask,secondTask);  
  
            return firstTask.join() + secondTask.join();  
        }  
  
    }  
}
```

```java
ForkJoinPool pool = new ForkJoinPool();  
  
CalculateStack task= new CalculateStack(0, Integer.MAX_VALUE);  
pool.submit(task);  
System.out.println(task.get());  
  
pool.awaitTermination(2, TimeUnit.SECONDS);  
pool.shutdown();
```