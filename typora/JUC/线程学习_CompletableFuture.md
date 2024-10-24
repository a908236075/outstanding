# CompletableFuture

## 前置铺垫

1. 并发:同一个实体上,同一个时刻,只有一件事情在发生.

2. 并行:不同实体的多个事件,同时在进行.

3. 用户线程:是指用户new thread() 出来的线程.

4. 守护线程,:为用户线程服务,在后台默默做一些服务,用户线程退出了,守护线程一会退出.设置Deamon 为true即可.

   1. ~~~java
      public class DaemonLearn {
          public static void main(String[] args) throws InterruptedException {
              Thread t1 = new Thread(() -> {
                  System.out.println(Thread.currentThread().getName());
                  while (true) {
      
                  }
              }, "t1");
              t1.setDaemon(true);
              t1.start();
              TimeUnit.SECONDS.sleep(3);
              System.out.println(Thread.currentThread().getName() + "is End");
          }
      }
      ~~~


## CompletableFuture作用

1. **异步并行**的计算功能.
2. 能够实现多线程/有返回/异步执行.
3. Callable和Runnable
   1. Callable有返回值,会抛出异常.
   2. Runnable没有返回值,不会抛出异常.

## CompletableFuture 技术演进

1. new Thread(Runnable run,String name); 如果想开启线程,需要有Runnable接口才行
2. RunnableFuture 满足了线程/异步执行,还缺一个返回值,即引出了FutureTask(Callable call). 

![](..\picture\JUC\FutureTask实现类.png)

## FutureTask代码实现

~~~java
public class FutureBaby {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask futureTask = new FutureTask<>(new CallSecond());
        Thread t1 = new Thread(futureTask, "t1");
        t1.start();
        futureTask.get();
    }
  
}
  // 因为我们想要一个返回值调用,所以实现callable方法
class CallSecond implements Callable<String> {
    @Override
    public String call() throws Exception {
        System.out.println("come in");
        return "Baby call back";
    }
}
~~~

## FutureTask 优缺点

1. 优点:多线程异步执行,显著提升执行速度.

   ~~~java
   public class FutureTest {
       public static void main(String[] args) throws ExecutionException, InterruptedException {
   
           long startTime = System.currentTimeMillis();
   //                test1();
           FutureTaskTest();
           long endTime = System.currentTimeMillis();
           System.out.println("耗时: " + (endTime - startTime) + "毫秒");
           System.out.println(Thread.currentThread().getName() + "......end");
       }
   
       private static void FutureTaskTest() throws InterruptedException, ExecutionException {
           ExecutorService threadPool = Executors.newFixedThreadPool(3);
           FutureTask<String> futureTask = new FutureTask<>(() -> {
               TimeUnit.MILLISECONDS.sleep(500);
               return "task1 over";
           });
           threadPool.submit(futureTask);
   
           FutureTask<String> futureTask2 = new FutureTask<>(() -> {
               TimeUnit.MILLISECONDS.sleep(300);
               return "task2 over";
           });
           threadPool.submit(futureTask2);
   
           FutureTask<String> futureTask3 = new FutureTask<>(() -> {
               TimeUnit.MILLISECONDS.sleep(200);
               return "task3 over";
           });
           threadPool.submit(futureTask3);
   //        System.out.println(futureTask.get());
           threadPool.shutdown();
       }
   
       public static void test1() throws InterruptedException {
   //        long startTime = System.currentTimeMillis();
           TimeUnit.MILLISECONDS.sleep(500);
           TimeUnit.MILLISECONDS.sleep(300);
           TimeUnit.MILLISECONDS.sleep(200);
           long endTime = System.currentTimeMillis();
   //        System.out.println("耗时: " + (endTime - startTime) + "毫秒");
           System.out.println(Thread.currentThread().getName() + "......end");
       }
   ~~~

   

2. 缺点:get()方法如果耗时,会导致阻塞,不停的询问是否执行完成,也会耗费CPU.

   ~~~java
   public class DisadvantageFuture {
   
       public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
           alwaysWait();
   //        isFinished();
   
       }
   private static void alwaysWait() throws InterruptedException, ExecutionException {
           FutureTask<String> task = new FutureTask<>(() -> {
               TimeUnit.SECONDS.sleep(5);
               System.out.println(Thread.currentThread().getName() + "执行了!!");
               return "success";
           });
           Thread t1 = new Thread(task, "t1");
           t1.start();
           // get会导致线程一直等待,直到任务获取到结果,或者直接设置一个最长等待时间,但是回报异常.
           task.get();
   //        task.get(2, TimeUnit.SECONDS);
           System.out.println(Thread.currentThread().getName() + "忙其他任务去了!!");
       }
   
    private static void isFinished() throws InterruptedException, ExecutionException {
           FutureTask<String> task = new FutureTask<>(() -> {
               System.out.println("执行了");
               TimeUnit.SECONDS.sleep(3);
               return "task over";
           });
           Thread thread = new Thread(task);
           thread.start();
           System.out.println(Thread.currentThread().getName() + "执行其他的任务去了");
   //        task.get(2, TimeUnit.SECONDS);
           while (true) {
               if (task.isDone()) {
                   System.out.println(task.get());
                   break;
               } else {
                   TimeUnit.SECONDS.sleep(1);
                   System.out.println("正在轮询.....");
               }
           }
       }
   }
   ~~~

   **所以要使用CompletableFuture!!!!**

## CompletableFuture的特点

1. 异步,多线程,有返回值.

2. 并行处理的任务,有结果依赖的关系,第二步骤需要前一个步骤的结果返回值.也有多个结果进行合并的情况.

3. 可以按照线程的负载,动态选择线程执行任务.

   ~~~java
   // 实现了Future 和complationStage
   public class CompletableFuture<T> implements Future<T>, CompletionStage<T> {}
   ~~~

## CompletableFuture静态的方法

~~~java
 /**
     * Returns a new CompletableFuture that is asynchronously completed
     * by a task running in the {@link ForkJoinPool#commonPool()} with
     * the value obtained by calling the given Supplier.
     *
     * @param supplier a function returning the value to be used
     * to complete the returned CompletableFuture
     * @param <U> the function's return type
     * @return the new CompletableFuture
     */
    public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier) {
        return asyncSupplyStage(asyncPool, supplier);
    }

    /**
     * Returns a new CompletableFuture that is asynchronously completed
     * by a task running in the given executor with the value obtained
     * by calling the given Supplier.
     *
     * @param supplier a function returning the value to be used
     * to complete the returned CompletableFuture
     * @param executor the executor to use for asynchronous execution
     * @param <U> the function's return type
     * @return the new CompletableFuture
     */
    public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier,
                                                       Executor executor) {
        return asyncSupplyStage(screenExecutor(executor), supplier);
    }

/**
     * Returns a new CompletableFuture that is asynchronously completed
     * by a task running in the {@link ForkJoinPool#commonPool()} after
     * it runs the given action.
     *
     * @param runnable the action to run before completing the
     * returned CompletableFuture
     * @return the new CompletableFuture
     */
    public static CompletableFuture<Void> runAsync(Runnable runnable) {
        return asyncRunStage(asyncPool, runnable);
    }

    /**
     * Returns a new CompletableFuture that is asynchronously completed
     * by a task running in the given executor after it runs the given
     * action.
     *
     * @param runnable the action to run before completing the
     * returned CompletableFuture
     * @param executor the executor to use for asynchronous execution
     * @return the new CompletableFuture
     */
    public static CompletableFuture<Void> runAsync(Runnable runnable,
                                                   Executor executor) {
        return asyncRunStage(screenExecutor(executor), runnable);
    }

~~~

1. supplyAsync和runAsync都各有两种方法,一个方法不需要传线程池,另一个需要传线程池,不传使用默认的**ForkJoinPool**线程池.
2. supplyAsync方法有返回值,而runAsync不需要返回值.

## 证明CompletableFuture能行

~~~java
public class CompletableFutureSecond {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
//        first();
        ExecutorService threadPool = Executors.newFixedThreadPool(3);
        try {
            CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(() -> {
                int result = ThreadLocalRandom.current().nextInt(10);
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("-----1秒钟后出现了结果!!!" + result);
                if (result > 5) {
                    int i = 10 / 0;
                }
                return result;
            }, threadPool).whenComplete((v, e) -> {
                if (e == null) {
                    System.out.println("-----计算完成,更新系统updateV:" + v);
                }
            }).exceptionally(e -> {
                e.printStackTrace();
                System.out.println("异常了: " + e.getCause() + "\t" + e.getMessage());
                return null;
            });
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdown();
        }

    }

    private static void first() throws InterruptedException, ExecutionException {
        CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(() -> {
            int result = ThreadLocalRandom.current().nextInt(10);
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("-----1秒钟后出现了结果!!!" + result);
            return result;
        });
        System.out.println(Thread.currentThread().getName() + "忙其他的线程去了!!!");
        System.out.println(completableFuture.get());
    }


}
~~~

## 处理数据和结果

1. 对结果进行处理

~~~java
public class CompletableFutureHandler {

    public static void main(String[] args) {
        // 因为handle代理异常作为参数,所以可以在方法体内进行处理,例如捕获,就可以不影响后续的步骤.
//        thenApplyMethod();
        handleMethod();
    }

    private static void handleMethod() {
        CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
            int i = 0;
            return i;
        }).handle((f, e) -> {
            return f + 2;
        }).handle((f, e) -> {
            return f + 3;
        }).whenComplete((f, e)->{
            System.out.println(f);
        }).exceptionally(e -> {
            e.printStackTrace();
            return -1;
        });
        future.join();
    }

    private static void thenApplyMethod() {
        CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
            int i = 0;
            return i;
        }).thenApply(f -> {
            return f + 2;
        }).thenApply(f -> {
            return f + 3;
        }).whenComplete((f, e) -> {
            System.out.println(f);
        }).exceptionally(e -> {
            e.printStackTrace();
            return -1;
        });

        future.join();
    }
}
~~~

2. 对结果进行消费

   1. 

   ~~~java
   public static void main(String[] args) {
       // thenAccept 直接将结果进行消费
           CompletableFuture<Void> completableFuture = CompletableFuture.supplyAsync(() -> {
               int i = 0;
               return i;
           }).handle((f, e) -> {
               return f + 2;
           }).handle((f, e) -> {
               return f + 3;
           }).thenAccept(f -> {
               System.out.println(f);
           });
       }
   ~~~

   2. 不同的消费方法
      1. ![](..\picture\JUC\CompletableFuture对结果进行消费.png)

## 选择线程池

1. 不传入线程池 thenRun()和thenRunAsync()都是使用默认的ForkJoinPool.
2. 传入线程池: 
   1. thenRun第一个任务使用自定义的线程池,第二个任务使用第一个任务的线程
   2. thenRunAsync()第一个任务使用的是传入的线程,第二个任务使用的是ForkJoinPool线程池里面的线程. 

