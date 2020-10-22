## lock（sysnchronized）

1. 概述

   1. > 通过使用lock同一对象，来控制锁的范围

2. 实现

   1. lock

   2. unlock

   3. trylock 

      1. > 在不阻塞线程的情况下去抢占锁

3. 种类

   1. ReentrantLock 

      1. >  重入锁:不需要再次抢占锁，而只是增加重入次数（对于同一对象的同一把锁），有效防止死锁

      2. > 乐观锁: 每次获取数据的时候，都不会担心数据被修改，所以每次获取数据的时候都不会进行加锁，适合读多写少

      3. > 公平锁和非公平锁

   2. ReentrantReadWriteLock 

      1. 名称
         1. 重入读写锁
      2. 种类包含
         1. readlock
         2. writelock
      3. 特性
         1. 读和读不互斥
         2. 读和写互斥
         3. 写和写互斥

4. 实现原理

   1. 锁的互斥 - 共享资源 - 记录锁的状态state，1为有锁状态，0为无锁状态
   2. 未获得锁的线程放到fifo双向队列，先进先出
   3. locksupport.park挂起线程，释放cpu资源，unpark唤醒指定线程
   4. 公平和非公平，能否插队
   5. 重入的特性（识别是否时同一个线程 threadid）某个地方存储线程id

5. 图解（aqs）

    ![img](https://raw.githubusercontent.com/LiangyangAlan/images/master/img/AQS.jpg)



6. 总结

   1. reentrantlock是非阻塞的，通过cas的原子替换，使得一个线程的阻塞与挂起不影响其他线程的阻塞
   2. lock.lock通过自旋，通过tryacquire进行cas原子操作，成功，获取锁，失败则cas操作更改状态，使得线程挂起
   3. cas 操作满足原子性，多个线程同时操作只有一个成功基于总线索和缓存索的机制，缓存一致性，保证数据在修改时的安全性
   4. 当线程过多时，不断循环cas操作，会带来一定的性能开销
   5. 和普通互斥锁的区别：普通互斥锁会唤醒头节点的下个节点，aqs队列则是唤醒头节点下个节点的**signal节点**



## Condition(条件控制)和阻塞队列

![](https://raw.githubusercontent.com/LiangyangAlan/images/master/img/condition阻塞.jpg)

Node结点是对每一个等待获取资源的线程的封装，其包含了需要同步的线程本身及其等待状态，如是否被阻塞、是否等待唤醒、是否已经被取消等。变量waitStatus则表示当前Node结点的等待状态，共有5种取值CANCELLED、SIGNAL、CONDITION、PROPAGATE、0。
- **CANCELLED**(1)：表示当前结点已取消调度。当timeout或被中断（响应中断的情况下），会触发变更为此状态，进入该状态后的结点将不会再变化。
- **SIGNAL**(-1)：表示后继结点在等待当前结点唤醒。后继结点入队时，会将前继结点的状态更新为SIGNAL。
- **CONDITION**(-2)：表示结点等待在Condition上，当其他线程调用了Condition的signal()方法后，CONDITION状态的结点将**从等待队列转移到同步队列中**，等待获取同步锁。
- **PROPAGATE**(-3)：共享模式下，前继结点不仅会唤醒其后继结点，同时也可能会唤醒后继的后继结点。
- **0**：新结点入队时的默认状态。


##  CountDownlatch

1. 流程
   1. `CountDownLatch countDownLatch = new CountDownLatch(1);`
      1. 设置state次数，在state=0之前阻塞
   2. `countDownLatch.await();`
      1. 实现线程的阻塞与join相似阻塞当前队列
   3. `countDownLatch.countDown();`
      1. state次数减一，当为0时，进行唤醒
      2. 实现线程的唤醒（全部唤醒）
      3. 唤醒的传递`setHeadAndPropagate(node, r)`
2. 特点
   1. 运用了共享锁的概念，即唤醒的线程都使用同一把锁
   2. 也是运用了aqs队列



## Semaphore(信号灯)

1. 流程
   1. `Semaphore semaphore = new Semaphore(1);`
      1. 设置令牌数（线程数），即只允许的线程数运行
   2. `semaphore.acquire();`
      1. 获得令牌继续执行
      2. 未获得令牌的进行阻塞
   3. `semaphore.release();`
      1. 释放令牌
2. 特点
   1. 和上面类似，只是表现形式不一样，都是控制线程的方案



## Atomic原子操作(安全性)

1. i++问题
   1. 由于cpu高速缓存的存在
   2. 在操做非原子指令会导致cpu的中断，导致多个指令无法满足同时成功或者同时失败
2. `i.decrementAndGet();`
   1. cas方法原子替换，实现原子性



