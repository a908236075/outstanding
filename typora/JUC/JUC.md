# JUC并发编程

## AQS原理以及源码

1. ReentrantLock--> Sync-(继承)->AbstractQueuedSynchronizer 抽象队列锁 以及ReentrantLock对应的关系.

   - ![](..\typora\picture\JUC\AQS与Lock的关系.png)

2. 加锁会导致阻塞,持有和释放锁通过AQS进行管理.底层是是队列和volatile修饰的int类型的state.

3. Node

   1. ~~~java
       static final class Node {
              /** Marker to indicate a node is waiting in shared mode */
              static final Node SHARED = new Node();
              /** Marker to indicate a node is waiting in exclusive mode */
              static final Node EXCLUSIVE = null;
         
              /** waitStatus value to indicate thread has cancelled */
              static final int CANCELLED =  1;
              /** waitStatus value to indicate successor's thread needs unparking */
              static final int SIGNAL    = -1;
              /** waitStatus value to indicate thread is waiting on condition */
              static final int CONDITION = -2;
              /**
               * waitStatus value to indicate the next acquireShared should
               * unconditionally propagate
               */
              static final int PROPAGATE = -3;
         
              /**
               * Status field, taking on only the values:
               *   SIGNAL:     The successor of this node is (or will soon be)
               *               blocked (via park), so the current node must
               *               unpark its successor when it releases or
               *               cancels. To avoid races, acquire methods must
               *               first indicate they need a signal,
               *               then retry the atomic acquire, and then,
               *               on failure, block.
               *   CANCELLED:  This node is cancelled due to timeout or interrupt.
               *               Nodes never leave this state. In particular,
               *               a thread with cancelled node never again blocks.
               *   CONDITION:  This node is currently on a condition queue.
               *               It will not be used as a sync queue node
               *               until transferred, at which time the status
               *               will be set to 0. (Use of this value here has
               *               nothing to do with the other uses of the
               *               field, but simplifies mechanics.)
               *   PROPAGATE:  A releaseShared should be propagated to other
               *               nodes. This is set (for head node only) in
               *               doReleaseShared to ensure propagation
               *               continues, even if other operations have
               *               since intervened.
               *   0:          None of the above
               *
               * The values are arranged numerically to simplify use.
               * Non-negative values mean that a node doesn't need to
               * signal. So, most code doesn't need to check for particular
               * values, just for sign.
               *
               * The field is initialized to 0 for normal sync nodes, and
               * CONDITION for condition nodes.  It is modified using CAS
               * (or when possible, unconditional volatile writes).
               */
              volatile int waitStatus;
       }
      ~~~

   2. Node的状态值代表的含义![](..\typora\picture\JUC\AQS中的Node属性值.png)

4. 公平锁与非公平锁

   1. ![](..\typora\picture\JUC\(非)公平锁.png)
   2. ![](..\typora\picture\JUC\公平与非获取锁的区别.png)
   3. 公平锁会优先让等待的队列中的node获取锁.

5. ReentrantLock可重入锁非公平锁的lock方法

   1. ~~~java
      final void lock() {
                 if (compareAndSetState(0, 1))
                     setExclusiveOwnerThread(Thread.currentThread());
                 else
                     acquire(1);
             }
       public final void acquire(int arg) {
             if (!tryAcquire(arg) &&
                 acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
                 selfInterrupt();
         }
      ~~~

   2. 锁主要由三个方法组成

      1. tryAcquire

         1. ~~~java
              final boolean nonfairTryAcquire(int acquires) {
                        final Thread current = Thread.currentThread();
                        int c = getState();
                        if (c == 0) {
                            if (compareAndSetState(0, acquires)) {
                                setExclusiveOwnerThread(current);
                                return true;
                            }
                        }
                        else if (current == getExclusiveOwnerThread()) {
                            int nextc = c + acquires;
                            if (nextc < 0) // overflow
                                throw new Error("Maximum lock count exceeded");
                            setState(nextc);
                            return true;
                        }
                        return false;
                    }
            ~~~

      2. addWaiter 

         1. 将node节点放入队列

      3. acquireQueued

         1. ~~~java
            final boolean acquireQueued(final Node node, int arg) {
                    boolean failed = true;
                    try {
                        boolean interrupted = false;
                        for (;;) {
                            final Node p = node.predecessor();
                            if (p == head && tryAcquire(arg)) {
                                setHead(node);
                                p.next = null; // help GC
                                failed = false;
                                return interrupted;
                            }
                            if (shouldParkAfterFailedAcquire(p, node) &&
                                parkAndCheckInterrupt())
                                interrupted = true;
                        }
                    } finally {
                        if (failed)
                            cancelAcquire(node);
                    }
                }
            
             private final boolean parkAndCheckInterrupt() {
                    LockSupport.park(this);
                    return Thread.interrupted();
                }
            
            ~~~

         2. 主要的作用是设置锁的状态. state=1  并将节点的waitStatus状态设置为-1 代表 已经准备好,随时可以调用.  


## 读锁与写锁

​	1. Lock.lock() 将所包含的业务进行串行,但是当有读取的时候,希望线程不阻塞,所有,创建了读锁和写锁,读读共享,读写互斥 .

```java
ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
lock.readLock().lock();
lock.readLock().unlock();
lock.writeLock().lock();
lock.writeLock().unlock();
```

## 邮戳锁

1. 当读线程非常多，写线程很少的情况下，很容易导致写线程“饥饿”，也就是线程负责读任务,但是有一个写任务,只能一直等待读任务完成后,才能进行.

2. 解决的办法是实现了一个队列,每次不管是读锁也好写锁也好,未拿到锁就加入队列,然后每次解锁队列头存储的线程节点获取锁,以此避免饥饿。

3. ~~~java
   class Point {
       private int x, y;
       final StampedLock sl = new StampedLock();
    
       //计算到原点的距离  
       double distanceFromOrigin() {
           // 乐观读
           long stamp = sl.tryOptimisticRead();
           // 读入局部变量，
           // 读的过程数据可能被修改
           int curX = x, curY = y;
           //判断执行读操作期间，
           //是否存在写操作，如果存在，
           //则sl.validate返回false
           if (!sl.validate(stamp)) {
               // 升级为悲观读锁
               stamp = sl.readLock();
               try {
                   curX = x;
                   curY = y;
               } finally {
                   //释放悲观读锁
                   sl.unlockRead(stamp);
               }
           }
           return Math.sqrt(curX * curX + curY * curY);
       }
   }
   ~~~

## Lock与Synchronized锁不同

![](.\picture\JUC\Lock与Synchronized锁区别.png)

## 线程池

### 实现原理

![](..\typora\picture\JUC\线程池实现原理.png)

### 重要的参数

![](..\typora\picture\JUC\线程池的重要参数.png)

## 其它知识

1. 重写Thread和重写Runable接口的区别
   1. 继承Thread类的，我们相当于拿出三件事即三个卖票10张的任务分别分给三个窗口，他们各做各的事各卖各的票各完成各的任务，因为MyThread继承Thread类，所以在new MyThread的时候在创建三个对象的同时创建了三个线程；
   2. 实现Runnable的， 相当于是拿出一个卖票10张得任务给三个人去共同完成，new MyThread相当于创建一个任务，然后实例化三个Thread，创建三个线程即安排三个窗口去执行。
2. 为什么有这么多锁
   1. 引入事务,使用Lock 或者 Synchronized
   2. 发现读的时候不需要阻塞,甚至读写的时候需要区别处理,所以引入了读写锁,并设置了可重入的特性.
   3. 一般读多写少,导致写锁饥饿的问题,引入了邮戳锁.

## CompletableFuture

### FutureTask

![](.\picture\JUC\FutureTask.png)

2. FutureTask 实现了 RunableFuture,构造方法还需要传递一个Callbale,所以他是回调/异步/多线程 都能具有的类.

### Future 的缺点

1. ~~~java
   public class DisadvantageFuture {
       public static void main(String[] args) throws ExecutionException, InterruptedException {
           FutureTask<String> task = new FutureTask<>(() -> {
               System.out.println("执行了");
               TimeUnit.SECONDS.sleep(3);
               return "task over";
           });
           Thread thread = new Thread(task);
           thread.start();
           System.out.println(Thread.currentThread().getName() + "执行其他的任务去了");
       //    task.get();
      //   task.get(2, TimeUnit.SECONDS);
            while (true) {
               if (task.isDone()) {
                   System.out.println(task.get());
                   break;
               }else {
                   TimeUnit.SECONDS.sleep(1);
                   System.out.println("正在轮询.....");
               }
           }
       }
   }
   ~~~

2. get()容易导致阻塞,一般建议放在程序的后面.

3. 轮询耗费CPU的资源.

### CompletableFuture

1. 两种创建CompletableFuture的方式

   1. ```java
      public class CompletableFutureFirst {
          public static void main(String[] args) throws ExecutionException, InterruptedException {
      //        runAsyncExecutorService();
              CompletableFuture<String> firstRunning = CompletableFuture.supplyAsync(() -> {
                  try {
                      TimeUnit.SECONDS.sleep(1);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
                  System.out.println("CompletableFutureFirst running")    ;
                  return "CompletableFutureFirst task over!!";
              });
      
              System.out.println(firstRunning.get());
          }
      
          private static void runAsyncExecutorService() throws InterruptedException, ExecutionException {
              ExecutorService executorService = Executors.newFixedThreadPool(3);
              CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(() -> {
                  System.out.println(Thread.currentThread().getName());
                  try {
                      TimeUnit.SECONDS.sleep(1);
                      System.out.println("CompletableFutureFirst run");
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }, executorService);
              System.out.println(completableFuture.get());
              executorService.shutdown();
          }
      
      }
      // runAsync 方法没有返回值  supplyAsync 方法有
      ```

2. 案例优化,比较FutureTask不需要轮询,就能获取结果

   1. ~~~java
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

   2. 接收结果后处理
   
      1. ~~~java
         public class CompletableFutureResult {
         
             public static void main(String[] args) {
                 CompletableFuture.supplyAsync(() -> 1)
                         .thenApply(a -> a + 2)
                         .whenComplete((ele, throwable) -> {
                     System.out.println(ele);
                     String s = throwable.getMessage() + ":" + throwable.getCause();
                     System.out.println(s);
                 });
             }
         }
         // thenApply 将结果进行返回
         // handle 正常的情况下 和thenApply一样,只有异常的时候,即使遇到了异常,也能执行之后的步骤.
         // handle 一般是必须要执行的步骤  例如关流
         ~~~
   
      2. thenRun,不需要上一步的结果,也没有返回结果,thenAccept 需要上一步结果,自己没有结果.thenApply 需要上一步结果,也有返回值.
   
      3. join和get一样都可以返回得到执行的结果,但是join不会报异常.
   
      4. thenRunAsync() 调用的时候,需要自定义线程池,才行实现多线程的调用.第一个任务,使用的自定义的线程池,之后的步骤使用的是CommonPool线程池.也就是默认的线程池.thenRun(),方法,只会调用上一个步骤的线程.
   
      5. 异步回调的时候需要传出我们自己的线程池.
   
      6. 线程池循环容易导致死锁,即 CompletableFuture.supplyAsync在调用 CompletableFuture.supplyAsync,当CommonPool线程打满,而子线程还在等待释放线程,而主线程又等待子线程执行完任务后在释放线程,循环等待,导致死锁.
   
         1. ~~~java
            public Object doGet() {
              ExecutorService threadPool1 = new ThreadPoolExecutor(10, 10, 0L, TimeUnit.MILLISECONDS, new ArrayBlockingQueue<>(100));
              CompletableFuture cf1 = CompletableFuture.supplyAsync(() -> {
              //do sth
                return CompletableFuture.supplyAsync(() -> {
                    System.out.println("child");
                    return "child";
                  }, threadPool1).join();//子任务
                }, threadPool1);
              return cf1.join();
            }
            ~~~
   
         2. 
   
      7. 
