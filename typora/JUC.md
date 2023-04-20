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
