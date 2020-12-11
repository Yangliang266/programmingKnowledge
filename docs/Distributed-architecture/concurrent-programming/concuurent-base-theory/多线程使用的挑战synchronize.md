## 多线程使用的挑战



### 多线程应用产生的问题

1. 数据的一致性

   > 多线程通过cpu的时间片切换运行导致共同的变量发生改变极易产生脏数据，导致线程之间的数据不一致

2. 线程不安全

   > 多个线程访问同一个共享对象  这个共享对象的状态发生错误，对象的结果与预想值发生不一致



### 解决方案

#### 1.synchonized的使用

1. 本质

   1. 非公平锁
   2. 悲观锁: 当一个线程获得锁时，后续线程都会处于阻塞状态，直到释放，才会重新竞争，适合写多读少的场景
   3. 共享资源

2. 存在意义

   1. > 加锁的情况下，会导致原有的并行线程变为串行，在保证数据一致性的情况下，会损失一定的性能

   2. > `synchonized`引入偏向锁，轻量级锁，重量级锁

   3. > 偏向锁和轻量级锁一定程度上使得在不加锁的情况下也能完成并发编程，提升性能

3. 控制粒度（锁范围）

   1. 修饰实例方法

      1. > 对一个对象中的此种方法多次使用，相互排斥，但对于多个对象使用自身的方法，互不干扰，属于对象级别

         ```java
         public synchronized void incr(){
             try {
                 Thread.sleep(1);
             } catch (InterruptedException e) {
                 e.printStackTrace();
             }
             count++;
         }
         ```

         

   2. 修饰静态方法

      1. > 对于整个类的此种方法进行粒度控制

         ```java
         public synchronized static void incr(){
             try {
                 Thread.sleep(1);
             } catch (InterruptedException e) {
                 e.printStackTrace();
             }
             count++;
         }
         ```

         

   3. 修饰代码块

      1. > 此种修饰控制粒度更为灵活既可以修饰实例方法也可修饰类级别

         ```java
         public static void incr(){
             synchronized (AtomicDemo.class) {
                 try {
                     Thread.sleep(1);
                 } catch (InterruptedException e) {
                     e.printStackTrace();
                 }
                 count++;
             }
         }
         ```

         ```java
         private Object object;
         
         public /*synchronized*/ static void incr(){
             synchronized (object) {
                 try {
                     Thread.sleep(1);
                 } catch (InterruptedException e) {
                     e.printStackTrace();
                 }
                 count++;
             }
         }
         ```

         

4. 锁的升级

   1. 偏向锁

      <img src="https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/java/concurrent/%E5%81%8F%E5%90%91%E9%94%81.png" style="zoom:80%;" />

   2. 轻量级重量级

      <img src="https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/java/concurrent/%E8%BD%BB%E9%87%8F%E7%BA%A7%E9%94%81.png" style="zoom:80%;" />

   3. 名词解释

      1. CAS 原子替换

         > 期望值和预期值是否一致，如果一致进行替换，否则替换失败

      2. Markword 锁标记存储位置

         > 轻量级锁标记存储在markword中即在堆区中的对象头中
         >
         > 重量级锁标记在虚拟机栈中创建的lockrecord中，指针指向markword的锁记录

      

      
   
      

## synchronized控制流程

![](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/java/concurrent/synchronized.jpg)