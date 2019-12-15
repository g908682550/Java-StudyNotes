# JUC（java.util.concurrent）

## 1.1进程/线程

线程的六种状态：

NEW

RUNNABLE

BLOCKED

WATTING

TIMED_WATTING

TERMINATED

## 1.2并发/并行

# 三个包

## java.util.concurrent

### java.util.concurrent.aomic

### java.util.concurrent.locks

~~~java
class Ticket //资源类=实例变量+实例方法
{
    private int number=30;

    Lock lock=new ReentrantLock();//Lock接口，可重入锁

    public void sale(){
        lock.lock();
        try{
            if(number>0){
                System.out.println(Thread.currentThread().getName()+"\t卖出第："+(number--)+"\t 还剩下： "+number);
            }
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            lock.unlock();
        }
    }

}
/*
题目：三个售票员    卖出       30张票
笔记： 如何编写企业级的多线程代码
    固定的编程套路+模板?

    1 在高内聚低耦合的前提下，线程     操作     资源类
        1.1 先创建一个资源类
        1.2 线程操作资源类

 */
public class SaleTicketDemo01 {
    public static void main(String[] args) {//主线程，一切程序的入口

        Ticket ticket=new Ticket();
//lambda表达式写法
        new Thread(()->{for(int i=1;i<=40;i++)ticket.sale();},"A").start();
        new Thread(()->{for(int i=1;i<=40;i++)ticket.sale();},"B").start();
        new Thread(()->{for(int i=1;i<=40;i++)ticket.sale();},"C").start();


        //Thread(Runnable target,String name)
//        new Thread(new Runnable() {//匿名内部类，new一个接口
//            @Override
//            public void run() {
//                for(int i=1;i<=40;i++){
//                    ticket.sale();
//                }
//            }
//        }, "A").start();
//
//        new Thread(new Runnable() {//匿名内部类，new一个接口
//            @Override
//            public void run() {
//                for(int i=1;i<=40;i++){
//                    ticket.sale();
//                }
//            }
//        }, "B").start();
//
//        new Thread(new Runnable() {//匿名内部类，new一个接口
//            @Override
//            public void run() {
//                for(int i=1;i<=40;i++){
//                    ticket.sale();
//                }
//            }
//        }, "C").start();

    }
}
~~~

# LambdaExpression

1、口诀：拷贝小括号，写死右箭头，落地大括号



2、接口有一个方法可称为函数式接口，该接口默认被注解@FunctionalInterface



3、在接口内default声明方法后可以为默认方法做具体实现



4、也可以写静态方法实现

​	

# 线程间通信

生产者消费者基础模型

```
题目：现在两个线程，可以操作初始值为零的一个变量，
实现一个线程对该变量加1，一个线程对该变量减1，
实现交替，来10伦，变量初始值为零

    1、高内聚低耦合前提下，线程操作资源类
    2、判断/干活/通知
    3、多线程交互中，必须要防止多线程的虚假唤醒，即（判断只用while，不能用if）
    4、注意标志位的修改和定位
```

~~~java
class AirConditioner{
    private int number=0;

    public synchronized void increment() throws InterruptedException {
        //1、判断
        while(number!=0) this.wait();
        //2、干活
        number++;
        System.out.println(Thread.currentThread().getName()+"\t"+number);
        //3、通知
        this.notifyAll();
    }

    public synchronized void decrement() throws InterruptedException
    {
        while(number==0) this.wait();
        number--;
        System.out.println(Thread.currentThread().getName()+"\t"+number);
        this.notifyAll();
    }
  
  //使用JUC
      public void increment() throws InterruptedException{
        lock.lock();
        try{
            while(number!=0) condition.await();
            //2、干活
            number++;
            System.out.println(Thread.currentThread().getName() + "\t" + number);
            //3、通知
            condition.signalAll();//this.notifyAll();
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            lock.unlock();
        }
    }

    public void decrement() throws InterruptedException{
        lock.lock();
        try{
            while(number==0) condition.await();//this.wait();
            //2、干活
            number--;
            System.out.println(Thread.currentThread().getName() + "\t" + number);
            //3、通知
            condition.signalAll();//this.notifyAll();
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            lock.unlock();
        }
    }
}
public class ThreadWaitNotifyDemo {
    public static void main(String[] args) {
        AirConditioner airConditioner=new AirConditioner();
        new Thread(()->{
            try {
                for(int i=0;i<10;i++)
                airConditioner.increment();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"A").start();

        new Thread(()->{
            try {
                for(int i=0;i<10;i++)
                    airConditioner.decrement();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"B").start();
    }
}
~~~

## 对进程的精准通知和精准唤醒

```
/**
 * 多线程之间按顺序调用,A->B->C
 * 三个线程启动后运行顺序如下：
 * AA打印5次，BB打印10次，CC打印15次之后重复，共循环10次
 */
```

~~~java
class ShareResouce {
    private int number=1;//1:A 2:B 3:C
    private Lock lock=new ReentrantLock();
    private Condition condition1=lock.newCondition();
    private Condition condition2=lock.newCondition();
    private Condition condition3=lock.newCondition();

    public void print5(){
        lock.lock();
        try{
            //判断
            while(number!=1){
                condition1.await();
            }
            //干活
            System.out.println("A打印了5次");
            //通知
            number=2;
            condition2.signal();
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            lock.unlock();
        }
    }

    public void print10(){
        lock.lock();
        try{
            //判断
            while(number!=2){
                condition2.await();
            }
            //干活
            System.out.println("B打印了10次");
            //通知
            number=3;
            condition3.signal();
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            lock.unlock();
        }
    }

    public void print15(){
        lock.lock();
        try{
            //判断
            while(number!=3){
                condition3.await();
            }
            //干活
            System.out.println("C打印了15次");
            //通知
            number=1;
            condition1.signal();
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            lock.unlock();
        }
    }

}
public class ThreadOrderAccess {

    public static void main(String[] args) {
        ShareResouce shareResouce=new ShareResouce();
        new Thread(()->{for(int i=0;i<10;i++) shareResouce.print5();},"A").start();
        new Thread(()->{for(int i=0;i<10;i++) shareResouce.print10();},"B").start();
        new Thread(()->{for(int i=0;i<10;i++) shareResouce.print15();},"C").start();

    }
}
~~~

# 八锁问题

~~~java
//资源类
class Phone
{
    public synchronized void sendEmail() throws Exception{
        try{ TimeUnit.SECONDS.sleep(4);} catch(InterruptedException e){e.printStackTrace();}
        System.out.println("sendEmail");
    }

    public synchronized void sendSMS() throws Exception{
        System.out.println("sendSMS");
    }

    public void hello(){
        System.out.println("---hello");
    }
}
~~~



```
/**
 *多线程8锁
 * 1、标准访问，请问先打印邮件还是短信？
 * 2、邮件方法暂停4秒中，请问先打印邮件还是短信？
 * 3、新增一个普通方法hello，请问先打印邮件还是hello?
 * 4、两部手机，请问先打印邮件还是短信
 * 5、两个静态同步方法，同一部手机，请问先打印邮件还是短信？
 * 6、两个静态同步方法，同一部手机，请问先打印邮件还是短信？
 * 7、一个普通同步方法，一个静态同步方法，一部手机，请问先打印邮件还是短信？
 * 8、一个普通同步方法，一个静态同步方法，两部手机，请问先打印邮件还是短信？
 */
```

 explain:

​	一个对象里面如果有多个synchronized方法，某一时刻内，只要一个线程去调用其中的一个synchronized方法了，其它的线程都只能等待，换句话说，某一时刻内，只能有唯一一个线程去访问这些synchronized方法，锁的是当前对象的this，被锁定后，其它的线程都不能进入到当前对象的其它synchronized方法。

​	加个普通方法后发现和同步锁无关，换成两个对象后，不是同一把锁了，情况立刻发生变化

​	都换成静态同步方法后，情况又变化

​	new this ,具体的一个对象

​	静态class，唯一的一个模板

​	所有的非静态同步方法用的都是同一把锁-实例对象

​	synchronized实现同步的基础：Java中的每一个对象都可以作为锁，具体表现为以下3种形式。（1）对于普通同步方法，锁是当前实例对象。（2）对于静态同步方法，锁是当前类Class对象。（静态方法通过Class类调用）（3）对于同步方法块，锁是synchronized括号里配置的对象。

当一个线程试图访问同步代码块时，它首先必须得到锁，退出或抛出异常必须释放锁。

也就是说一个实例对象的非静态同步方法获取锁后，该实例对象的其它非静态同步方法必须等待获取锁的方法释放锁后才能调用

可是别的实例对象的非静态同步方法因为跟该实例对象的非静态同步方法用的是不同的锁，所以无需等待该实例对象已经获取锁的非静态同步方法释放锁就可以获取他们自己的锁

所有的静态同步方法用的也是同一把锁-类对象本身

这两把锁是两个不同的对象，所以静态同步方法与非静态同步方法之间是不会有竞争条件的。但是一旦一个静态同步方法获取锁后，其他的静态同步方法都必须等待该方法释放锁后才能获取锁，而不管是同一个实例对象的静态同步方法还是不同实例对象的同步方法，只要它们是同一个类的实例对象。

# 集合不安全

## List不安全

~~~java
/**
 *题目：请举例说明集合类不安全
 * 1、故障现象
 *  java.util.ConcurrentModificationException
 * 2、导致原因
 *
 * 3、解决方案
 *    3.1 Vector
 *    3.2 Collection.synchronizedList(new ArrayList<>());
 *    3.3 CopyOnWriteArrayList
 * 4、优化建议
 */
public class NotSafeDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        for(int i=0;i<=30;i++){
            new Thread(()->{
                list.add(UUID.randomUUID().toString().substring(0,8));
                System.out.println(list);
            },String.valueOf(i)).start();
        }
    }
}
~~~

写时复制

CopyOnWrite容器即写时复制的容器。往一个容器添加元素的时候，不直接往当前容器Object[] 添加，而是先将当前容器Object[]进行Copy，复制出一个新的容器Object[] newElements，然后往新的容器Object[] newElements里添加元素，添加完元素之后，再将原容器的引用指向新的容器 setArray(newElements)；这样做的好处是可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素，所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器

~~~java
    public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
~~~

##  Set不安全

CopyOnWriteArraySet；

仍使用写时复制

## Map不安全

ConcurrentHashMap;

# 获得多线程的方式

1、继承Thread类

2、实现Runnable接口

3、实现Callable接口

Callable接口与runnable接口区别：

（1）是否有返回值（返回值为泛型）

（2）是否抛异常

（3）落地方法不一样，一个是run，一个是call

new Thread（Runnable，String）

Runnable接口有子接口RunableFuture

FutrueTask类实现了Runnable接口

同时FutrueTask类有new FutrueTask(Callable<>()）;

可new Thread（FutrueTask，String）来创建实现了Callable接口的线程

# 三个常用多线程类

## CountDownLatch

CountDownLatch countdownlatch=new CountDownLatch(int);

countDownLatch.countDown();

countDownLatch.await();

原理：

CountDownLatch主要有两个方法，当一个或多个线程调用await方法时，这些线程会阻塞。其它线程调用countDown方法会将计数器减1（调用countDown方法的线程不会阻塞）。当计数器的值变为0时，因await方法阻塞的线程会被唤醒，继续执行

~~~java
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch=new CountDownLatch(5);
        for (int i = 0; i <5 ; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName() + "退出");
                countDownLatch.countDown();
            },String.valueOf(i)).start();
        }
        countDownLatch.await();
        System.out.println(Thread.currentThread().getName()+"退出");
    }
~~~



## CyclicBarrier

CyclicBarrier cyclicBarrier=new CyclicBarrier(int parties,Runnable barrierAction)

cyclicBarrier.await()

CyclicBarrier的字面意思是可循环（Cyclic）使用的屏障（Barrier）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。线程进入屏障通过CyclicBarrier的await()方法。

~~~java
    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier=new CyclicBarrier(7,()->{
            System.out.println("最后执行");
        });
        for (int i = 0; i <7 ; i++) {
            final int temp=i;
            new Thread(()->{
                System.out.println("第"+temp+"子线程");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            },String.valueOf(i)).start();
        }
    }
~~~



## SemaphoreDemo

Semaphore semaphore=new Semaphore(int); 规定资源总数

semaphore.acquire();资源-1

semaphore.release();资源+1



acquire（获取） 当一个线程调用acquire操作时，它要么通过成功获取信号量（信号量减1），要么一直等下去，直到有线程释放信号量，或超时。

release（释放）实际上会将信号量的值加1，然后唤醒等待的线程。

信号量主要用于两个目的，一个是用于多个共享资源的互斥使用，另一个用于并发线程数的控制。

~~~java
public static void main(String[] args) {
        Semaphore semaphore=new Semaphore(2);
        for (int i = 0; i <5 ; i++) {
            new Thread(()->{
                try{
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName()+"进来了");
                }catch (Exception e){
                    e.printStackTrace();
                }finally {
                    semaphore.release();
                    System.out.println(Thread.currentThread().getName()+"出去了");
                }
            },String.valueOf(i)).start();
        }
    }
~~~



# ReadWriteLock

多个线程同时读一个资源类没有任何问题，所以为了满足并发量，读取共享资源应该可以同时进行

但是

如果有一个线程想去写共享资源，就不应该再有其它线程可以对该资源进行读或写

总结：

读-读能共存

读-写不能共存

写-写不能共存

```java
ReadWriteLock readWriteLock=new ReentrantReadWriteLock();
readWriteLock.writeLock().lock();
readWriteLock.writeLock().unlock();

readWriteLock.readLock().lock();
readWriteLock.readLock().unlock();
```

# BlockingQueue

## 介绍

当队列是空的，从队列中获取元素的操作将会被阻塞
当队列是满的，从队列中添加元素的操作将会被阻塞

试图从空的队列中获取元素的线程将会被阻塞，直到其他线程往空的队列插入新的元素

试图向已满的队列中添加新元素的线程将会被阻塞，直到其他线程从队列中移除一个或多个元素或者完全清空，使队列变得空闲起来并后续新增

## 用处

在多线程领域：所谓阻塞，在某些情况下会挂起线程（即阻塞），一旦条件满足，被挂起的线程又会自动被唤起

为什么需要BlockingQueue

好处是我们不需要关心什么时候需要阻塞线程，什么时候需要唤醒线程，因为这一切BlockingQueue都给你一手包办了

## 种类

ArrayBlockingQueue：由数组结构组成的有界阻塞队列。

LinkedBlockingQueue：由链表结构组成的有界（但大小默认值integer.MAX_VALUE）阻塞队列。

PriorityBlockingQueue：支持优先级排序的无界阻塞队列。

DelayQueue：使用优先级队列实现的延迟无界阻塞队列。

SynchronousQueue：不存储元素的阻塞队列，也即单个元素的队列。

LinkedTransferQueue：由链表组成的无界阻塞队列。

LinkedBlockingDeque：由链表组成的双向阻塞队列。

## 常用方法

| 方法类型 | 抛出异常      | 特殊值      | 阻塞     | 超时                 |
| ---- | --------- | -------- | ------ | ------------------ |
| 插入   | add(e)    | offer(e) | put(e) | offer(e,time,unit) |
| 移除   | remove()  | poll()   | take() | poll(time,unit)    |
| 检查   | element() | peek()   | 不可用    | 不可用                |

| 种类   | 描述                                       |
| ---- | ---------------------------------------- |
| 抛出异常 | 阻塞队列满时，add方法抛异常；空时，remove方法抛出异常          |
| 特殊值  | offer方法，成功true，失败false。poll方法成功返回元素，失败返回null |
| 一直阻塞 | 队列满时，生产者线程put元素，队列会阻塞直到put数据or响应中断退出；   队列为空，消费者线程take元素，队列会阻塞消费者线程直到队列可用 |
| 超时退出 | 当阻塞队列满时，队列会阻塞生产者线程一定时间，超过限时后生产者线程会退出     |

# ThreadPool

## 线程池的优势

线程池做的工作只要是控制运行的线程数量，处理过程中将任务放入队列，然后在线程创建后启动这些任务，如果线程数量超过了最大数量，超出数量的线程排队等候，等其他线程执行完毕，再从队列中取出任务来执行。


它的主要特点为：**线程复用**;**控制最大并发数**;**管理线程**。

第一：降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的销耗。
第二：提高响应速度。当任务到达时，任务可以不需要等待线程创建就能立即执行。
第三：提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会销耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

## 线程池的使用

Executors.newFixedThreadPool(int) 

执行长期任务性能好，创建一个线程池，一池有N个固定的线程，有固定线程数的线程



Executors.newSingleThreadExecutor()

一个任务一个任务的执行，一池一线程



Executors.newCachedThreadPool()

执行很多短期异步任务，线程池根据需要创建新线程，但在先前构建的线程可用时将重用它们。可扩容，遇强则强



threadPoll.execute(Runnable)使用

## ThreadPoolExecutor底层原理

~~~java
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
      
      
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
      
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
~~~

## 线程池的重要参数

~~~java
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
~~~



1、corePoolSize：线程池的常驻核心线程数

2、maximumPoolSize: 线程池中能容纳同时执行的最大线程数，此值必须大于等于1

3、keepAliveTime：多余的空闲线程的存活时间。当前池中线程数量超过corePoolSize时，当空闲时间达到keepAliveTime时，多余线程会被销毁直到只剩下corePoolSize个线程为主

4、unit：keepAliveTime的单位

5、workQueue：任务队列，被提交但尚未被执行的任务

6、threadFactory：表示生成线程池中工作线程的线程工厂用于创建线程，**一般默认的即可**

7、handler：拒绝策略，表示当队列满了，并且工作线程大于等于线程池的最大线程数（maximumPoolSize）时如何来拒绝请求执行的runnable的策略

## 线程池底层工作原理

1、在创建了线程池后，开始等待请求。
2、当调用execute()方法添加一个请求任务时，线程池会做出如下判断：
  2.1如果正在运行的线程数量小于corePoolSize，那么马上创建线程运行这个任务；
  2.2如果正在运行的线程数量大于或等于corePoolSize，那么将这个任务放入队列；
  2.3如果这个时候队列满了且正在运行的线程数量还小于maximumPoolSize，那么还是要创建非核心线程立刻运行这个任务；
  2.4如果队列满了且正在运行的线程数量大于或等于maximumPoolSize，那么线程池会启动饱和拒绝策略来执行。
3、当一个线程完成任务时，它会从队列中取下一个任务来执行。
4、当一个线程无事可做超过一定的时间（keepAliveTime）时，线程会判断：
    如果当前运行的线程数大于corePoolSize，那么这个线程就被停掉。
    所以线程池的所有任务完成后，它最终会收缩到corePoolSize的大小。



线程池不允许使用Executors去创建，而是通过ThreadPoolExecutor的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险

## 线程池的拒绝策略

等待队列已经满了，再也塞不下新任务了。

同时， 线程池中的max线程也达到了，无法继续为新任务服务。

这个时候我们就需要拒绝策略机制合理的处理整个问题

四种拒绝策略：

AbortPolicy（默认）：直接抛出RejectExecutionException异常

CallerRunsPolicy：“调用者运行”一种调节机制，该策略既不会抛弃任务，也不会抛出异常，而是将某些任务回退到调用者，从而降低新任务的流量。

DiscardPolicy：该策略默默地丢弃无法处理的任务，不予任何处理也不抛出异常，如果允许任务丢失，这是最好的一种策略。

DiscardOldestPolicy：抛弃队列中等待最久的任务，然后把当前任务加入队列中尝试再次提交当前任务

# 函数式接口

java.util.funtion

java内置四大函数式接口

| 函数式接口                 | 参数类型 | 返回类型    | 用途                                       |
| --------------------- | ---- | ------- | ---------------------------------------- |
| Consumer<T><br>消费型接口  | T    | void    | 对l类型为T的对象应用操作<br>包含方法:void accept(T t)   |
| Supplier<T> <br>供给型接口 | 无    | T       | 返回类型为T的对象<br>包含方法:T get()                |
| Funtion<T,R><br>函数型接口 | T    | R       | 对类型为T的对象应用操作，并返回结果。结果是R类型的对象。包含方法:R apply(T t) |
| Predicate<T><br>断定型接口 | T    | boolean | 确定类型为T的对象是否满足某约束，并返回boolean值，包含方法boolean test(T t) |

# Java Stream流式计算

java.util.stream

流是数据渠道，用于操作数据源（集合、数组等）所生成的元素序列。

“集合讲的是数据，流讲的是计算！”

## 特点

Stream不会存储元素

Stream不会改变源对象，相反，他们会返回一个持有结果的新Stream

Stream操作是延迟执行的。这意味着他们会等到需要结果的时候才执行。

## 怎么用

创建一个Stream：一个数据源（数组、集合）

中间操作：一个中间操作，处理数据源数据

终止操作：一个终止操作，执行中间操作链，产生结果



即 **源头** --->**中间流水线**--->**结果**

## 实例

~~~java
/**
 * 题目：按照给出数据。找出同时满足以下条件的用户
 * 偶数ID其年龄大于24且用户名转为大写且用户名字倒排序只输出一个用户名字
 */
public class MyThreadPoolDemo {
    public static void main(String[] args) {
        User u1=new User(11,"a",23);
        User u2=new User(12,"b",24);
        User u3=new User(13,"c",22);
        User u4=new User(14,"d",28);
        User u5=new User(16,"e",26);
        List<User>  list= Arrays.asList(u1,u2,u3,u4,u5);

        list.stream().filter(u->{return u.getId()%2==0;})
                .filter(u->{return u.getAge()>24;})
                .map(u->{return u.getName().toLowerCase();})
                .sorted((s1,s2)->{return s2.compareTo(s1);})
                .limit(1)
                .forEach(System.out::println);
    }
}
~~~

# ForkJoin

Fork：把一个复杂任务进行分拆，大事化小
Join：把分拆任务的结果进行合并



ForkJoinPool     分支合并池    类比=>   线程池

ForkJoinTask     类比=>   FutureTask 

~~~java
class MyTask extends RecursiveTask<Integer>{

    private static final Integer ADJUST_VALUE=10;
    private int begin;
    private int end;
    private int result;

    public MyTask(int begin, int end) {
        this.begin = begin;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        if((end-begin)<=ADJUST_VALUE){
            for(int i=begin;i<=end;i++){
                result+=i;
            }
        }else{
            int middle=(end+begin)>>>1;
            MyTask task01=new MyTask(begin,middle);
            MyTask task02=new MyTask(middle+1,end);
            task01.fork();
            task02.fork();
            result=task01.join()+task02.join();
        }
        return result;
    }
}

public class ForkJoinDemo {
    public static void main(String[] args) throws Exception {
        MyTask myTask=new MyTask(0,100);
        ForkJoinPool threadPool=new ForkJoinPool();

        ForkJoinTask<Integer> forkJoinTask=threadPool.submit(myTask);

        System.out.println(forkJoinTask.get());

        threadPool.shutdown();
    }
}
~~~

# 异步回调

~~~java
public class CompletableFutureDemo {
    public static void main(String[] args) throws Exception{
        CompletableFuture<Void> completableFuture=CompletableFuture.runAsync(()->
        {
            System.out.println(Thread.currentThread().getName()+"没有返回值");
        });

        //异步回调
        CompletableFuture<Integer> completableFuture1=CompletableFuture.supplyAsync(()->
        {
            System.out.println(Thread.currentThread().getName()+"有返回值");
            return 1024;
        }
        );

        completableFuture1.whenComplete((t,u)->{
            System.out.println("t:"+t);
            System.out.println("u:"+u);
        }).exceptionally(f->{
            System.out.println("exception"+f.getMessage());
            return 4444;
        }).get();
    }
}
~~~


