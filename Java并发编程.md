>Java并发
- [线程](#线程)
  - [创建线程](#创建线程)
  - [创建线程续](#创建线程续)
  - [启动方法](#启动方法)
  - [线程的停止](#线程的停止)
  - [线程状态](#线程状态)
  - [线程状态之间的转化](#线程状态之间的转化)
  - [阻塞的概念](#阻塞的概念)
  - [并发级别](#并发级别)
- [Thread和Object类的重要方法](#thread和object类的重要方法)
  - [wait, notify, notifyAll 方法详解](#wait-notify-notifyall-方法详解)
  - [手写生产者消费者设计模式](#手写生产者消费者设计模式)
  - [为什么这几个方法在Object方法里，而sleep在Thread里](#为什么这几个方法在object方法里而sleep在thread里)
  - [sleep方法详解](#sleep方法详解)
  - [wait/notify、sleep的异同](#waitnotifysleep的异同)
  - [join方法](#join方法)
  - [yield方法](#yield方法)
  - [park方法](#park方法)
  - [其他](#其他)
- [线程池](#线程池)
  - [线程池的创建](#线程池的创建)
    - [构造参数](#构造参数)
    - [线程的添加规则](#线程的添加规则)
    - [线程池的调用逻辑](#线程池的调用逻辑)
    - [常用默认线程池](#常用默认线程池)
  - [线程数量的设计](#线程数量的设计)
  - [停止线程池的方法](#停止线程池的方法)
  - [拒绝任务的策略](#拒绝任务的策略)
  - [线程池的组成与实现原理](#线程池的组成与实现原理)
- [并发工具类](#并发工具类)
  - [常见的并发控制工具类](#常见的并发控制工具类)
  - [CountDownLatch](#countdownlatch)
  - [Semaphore](#semaphore)
  - [Condition](#condition)
  - [CyclicBarrier](#cyclicbarrier)
- [常用的并发容器](#常用的并发容器)
  - [淘汰的并发容器](#淘汰的并发容器)
  - [ConcurrentHashMap](#concurrenthashmap)
  - [并发队列](#并发队列)
  - [阻塞队列BlockingQueue](#阻塞队列blockingqueue)
  - [非阻塞队列ConcurrentLinkedQueue](#非阻塞队列concurrentlinkedqueue)
  - [CopyOnWriteArrayList](#copyonwritearraylist)
- [并发中的锁](#并发中的锁)
  - [synchronized](#synchronized)
  - [Lock接口](#lock接口)
  - [Reentrantlock源码解析](#reentrantlock源码解析)
    - [构造](#构造)
    - [可重入机制](#可重入机制)
    - [Lock接口中的方法](#lock接口中的方法)
  - [乐观锁与悲观锁](#乐观锁与悲观锁)
  - [可重入锁与非可重入锁](#可重入锁与非可重入锁)
  - [公平与非公平](#公平与非公平)
  - [共享锁与排他锁](#共享锁与排他锁)
    - [读写锁](#读写锁)
  - [自旋锁和阻塞锁](#自旋锁和阻塞锁)
  - [中断锁与不可中断](#中断锁与不可中断)
  - [锁优化（JVM帮助实现）](#锁优化jvm帮助实现)
- [CAS原理](#cas原理)
  - [应用场景：](#应用场景)
  - [缺点](#缺点)
- [ThreadLocal类](#threadlocal类)
- [原子类](#原子类)
  - [AtomicInteger](#atomicinteger)
  - [AtomicIntegerArray](#atomicintegerarray)
  - [AtomicReference](#atomicreference)
  - [AtomicIntegerFiledUpdate](#atomicintegerfiledupdate)
  - [LongAdder](#longadder)
- [final不变性](#final不变性)
  - [不变性与final的关系](#不变性与final的关系)
- [Runnable与Callable接口](#runnable与callable接口)
  - [Runnable的缺陷](#runnable的缺陷)
  - [Callable 接口](#callable-接口)
  - [Future类](#future类)
  - [Callable和Future的关系](#callable和future的关系)
  - [重要方法](#重要方法)
    - [方法一：继承Callable<>类](#方法一继承callable类)
    - [方法二：lambda表达式](#方法二lambda表达式)
  - [JDK中的Future](#jdk中的future)
  - [用FutureTask创建Future](#用futuretask创建future)
  - [Future的注意点](#future的注意点)
- [AQS](#aqs)
  - [三大核心部分之state](#三大核心部分之state)
  - [三大核心部分之FIFO队列](#三大核心部分之fifo队列)
  - [三大核心部分之获取/释放](#三大核心部分之获取释放)
  - [AQS对资源的共享方式](#aqs对资源的共享方式)
- [并发存在的问题](#并发存在的问题)
  - [死锁](#死锁)
    - [银行家算法？](#银行家算法)
    - [发生死锁的条件](#发生死锁的条件)
    - [实际工程中如何避免死锁](#实际工程中如何避免死锁)
    - [以哲学家就餐问题为例](#以哲学家就餐问题为例)
    - [解决方案](#解决方案)
    - [有效避免的方法：](#有效避免的方法)
  - [活锁](#活锁)
  - [饥饿](#饥饿)
- [Java 内存模型——底层原理](#java-内存模型底层原理)
  - [什么是底层原理](#什么是底层原理)
  - [JVM内存结构、Java内存模型、Java对象模型](#jvm内存结构java内存模型java对象模型)
  - [JVM内存结构](#jvm内存结构)
  - [Java 对象模型](#java-对象模型)
  - [JMM内存模型](#jmm内存模型)
    - [有序性](#有序性)
    - [可见性](#可见性)
      - [JMM解决可见性的方法](#jmm解决可见性的方法)
  - [Happens-Before原则](#happens-before原则)
  - [一个例子](#一个例子)
  - [volatile关键字](#volatile关键字)
    - [使用场景](#使用场景)
  - [原子性](#原子性)
  - [单例模式写法，单例与并发的关系。](#单例模式写法单例与并发的关系)
- [线程安全](#线程安全)
  - [线程安全问题：](#线程安全问题)
  - [竞争条件冲突及分析](#竞争条件冲突及分析)
  - [逸出](#逸出)
  - [需要考虑线程安全的情况](#需要考虑线程安全的情况)
- [性能问题](#性能问题)
  - [调度：上下文切换](#调度上下文切换)
  - [协作：线程调度](#协作线程调度)
- [其他](#其他-1)
# 线程
**线程是操作系统能够进行运算调度的最小单位**.大部分情况下，**它被包含在进程之中，是进程中的实际运作单位。也就是说一个进程可以包含多个线程， 因此线程也被称为轻量级进程。**


* Q： java线程与操作系统线程的对应关系？
  A: 对于一个普通的应用程序，创建线程可以通过：使用内核线程（1：1）实现，使用用户进程(1:N)实现，使用用户线程+轻量级线程混合（N:M）.

    线程的实现不是JVM的规范所约束的，因此各家不太一样。hotspot现在一般都是采用直接映射一个操作系统的原生线程1：1创建。

    线程的调度一般就是协同式或者抢占式。协同就是线程自己控制调度，在完成自己的工作就会自行切换线程。不存在同步问题，lua的协程。但是不好控制bug。

    线程之间的切换时需要成本的。

    内核线程的调度成本主要来自于用户态和核心态的状态切换，主要是状态切换时候的响应中断，保护和恢复现场的成本。
## 创建线程
// 以下两个方法是正路
* 继承`Thread`类创建线程。要重写子方法`run()`，创建线程对象，调用`start`方法启动线程。(一般会创建一个静态内部类实现继承Thread方法)
```java
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("创建线程成功");
            }
        });
        thread.start();
```

升级：可以继承多个接口，且开销小
* 实现`Runnable`接口。首先子类继承接口并实现`run()`方法，然后分别创建`Runnable`子类实例（创建两个实现 Runnable 实现类的实例），以`Runnable`子类实例为对象，创建线程并启动。

```java
/**
 * @author chenzuoZhang
 */
public class RunnableDemo1 implements Runnable {

    private int i = 5;

    @Override
    public void run() {
        while (i > 0) {
            System.out.println(Thread.currentThread().getName() + " i = " + i);
            i--;
        }
    }

    public static void main(String[] args) {
        // 创建两个实现 Runnable 实现类的实例
        RunnableDemo1 runnableDemo1 = new RunnableDemo1();
        RunnableDemo1 runnableDemo2 = new RunnableDemo1();
        // 创建两个线程对象
        Thread thread1 = new Thread(runnableDemo1, "线程1");
        Thread thread2 = new Thread(runnableDemo2, "线程2");
        // 启动线程
        thread1.start();
        thread2.start();
    }

}

```

升级：继承 `Thread` 类和实现` Runnable` 接口这两种创建线程的方式都没有返回值。
* 实现`Callable`接口。首先创建`Callable`的实现类（一般是静态内部类），实现`call()`方法，这个方法是有返回体的（注意泛型）。创建`Callable`的实例，并且用`FutureTask`类来包装。以`FutureTask`为对象创建线程并启动。调用`FutureTask`对象的`get()`方法获得线程结束以后的返回值。

```java
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

/**
 * @author colorful@TaleLin
 */
public class CallableDemo1 {

    static class MyThread implements Callable<String> {

        @Override
        public String call() { // 方法返回值类型是一个泛型，在上面 Callable<String> 处定义
            return "我是线程中返回的字符串";
        }

    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // 常见实现类的实例
        Callable<String> callable = new MyThread();
        // 使用 FutureTask 类来包装 Callable 对象
        FutureTask<String> futureTask = new FutureTask<>(callable);
        // 创建 Thread 对象
        Thread thread = new Thread(futureTask);
        // 启动线程
        thread.start();
        // 调用 FutureTask 对象的 get() 方法来获得线程执行结束后的返回值
        String s = futureTask.get();
        System.out.println(s);
    }

}

```
## 创建线程续
Orale文档实际上只给出了两种方法。**一个是继承Thread类，重写run方法。另一个是实现Runnable接口的Run方法，并把Runnable实例传给Thread类**。

采用实现Runnable 方法更具有优势，
1. 从代码架构上可以解耦，一个是具体的任务也就是Run方法内部，另一个是与线程生命周期相关的内容等。
2. Thread方法每次需要新建一个线程，新建线程会有损耗高，无法重复利用同一个线程。 
3. 可以实现多个接口。

## 启动方法

采用`new Thread(runnable).star()`，这个方法会调用`star()`方法，**这个方法的核心是首先检查线程状态否则抛出异常，然后执行`star0`这个方法。这个方法会调用run方法执行。**

错误的方法是直接调用run方法，这样只是在主线程里面执行了。无法经历线程的各个周期。

## 线程的停止
停止线程的方法：`interrupt`会中断，但是不会强制。但是线程本身是最高的权限，保证不会被打断引发安全问题。

注意在sleep中发生中断并且被`catch`之后会清除中断标记位。之后无法用`Thread.curruntThread.isInterrupted()`检测到中断的发生。修改建议是，在catch语句中再次设置中断，`Thread.currentThread().interrupt();`。

处理中断的方法：需要请求方（发出信号），被停止方（时时检查），子方法配合（抛出中断）
1. 在子方法中抛出`throws`中断，留给更高层次的函数去检查
2. 子方法采用try-catch捕获了中断，重置中断，并且在高层次的函数中设计检查中断位，进行处理。

**错误做法**：子函数吞了中断请求，高层函数无法处理。

响应中断的方法：中断信号到了可以处理或者感知。
1. Object.wait()
2. Thread.sleep()
3. Thread.join()
4. java.util.concurrent.BlockingQueue.take()/put()
5. java.util.concurrent.locks.Lock.lockInterruptibly()
6. CountDownLatch.await()
7. CyclicBarrier.await()
8. Exchager.exchange()
9. Io相关操作

**错误的线程停止方法**：
1. 调用弃用Stop方法，会引起程序错乱。虽然stop方法会释放所有的监视器。
2. 使用弃用的suspend ，这个方法会被挂起，但是不会释放锁。容易导致死锁。
3. 采用置位volatile的方法。但是，这个方法在陷入阻塞时，无法中断线程。

**辨析**：
* static boolean interrupted() 这是一个静态方法，是当前执行类的。会获取中断标志位并且重置。
* boolean isInterrupted() 这个方法是需要被实例调用的，返回实例线程的中断标志。
* 
## 线程状态
六个新状态：New, Runnable, Blocked, Waiting, Timed Waiting, Terminated
* New：已创建但是还没启动的方法
* Runnable：调用`start`方法之后，一定进入这一状态，可运行的。**即使没有拿到资源，也是可运行的**
* Blocked：进入被`synchronized`的代码块或者方法修饰的。其他的不是。
* Waiting：没有设置时间参数的`a.wait()`或者`thread.join()`方法。
* Timed_Waiting：带有时间案数的wait方法
* Terminated：正常执行完结，或者发生未被捕获的异常抛出异常。

![* RUNNING](https://img-blog.csdnimg.cn/20200802205721626.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)
## 线程状态之间的转化
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200817101809236.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70#pic_center)

另外从图解Java并发设计模式中发现一幅图也清晰。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200904161850316.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70#pic_center)


## 阻塞的概念
一般情况下，把`Blocked `，`Waiting`，`Timed_waiting`都称为阻塞。

## 并发级别
1. 阻塞：释放资源之前，当前线程无法前进
2. 无饥饿：低优先级的线程无法取得资源
3. 无障碍：多个线程都可以进入临界区，发生冲突就回滚
4. 无锁：类似与CAS的操作。
5. 无等待：全部的线程都可以在有限步以内离开。

# Thread和Object类的重要方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200817153326287.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70#pic_center)

## wait, notify, notifyAll 方法详解
是`Object`类的方法。因此是对象的方法。**调用实例的`wait()`方法时候，线程被挂起，释放锁。**

阻塞阶段：获取了对象的monitor锁，进入休眠
唤醒方法：
* 另一个线程调用这个对象的`notify()`方法且刚好唤醒的是本线程；
* 另一个线程调用这个对象的`notifyAll()`方法；
* 过了`wait()`等待的时间；
* 等待的过程中调用`interrupt()`也会被唤醒；

遇到中断：抛出异常，释放锁。

特点性质：用必须先拥有monitor、只能唤醒一个、属于Object类、考虑同时持有多个锁的情况

## 手写生产者消费者设计模式
需要实现一个public类，三个单独的类，分别实现仓库（构造器：构造仓库，方法：get，take），消费者模式（构造器，继承Runnable重写run，调用take方法），生产者模式（构造器，继承Runnable，调用put方法）


思路是：首先一个启动类，需要构造仓库实例，消费者实例，生产者实例。并且新建两个线程分别执行两个实例。

消费者实例和生产者实例都需要实现`Runnable`接口，重写`run()`方法。并且需要有一个仓库作为实例传入。`run()`方法就是调用仓库的get和put方法。

仓库类是关键，首先构造函数需要得到一个长度有限的列表实例，然后需要写put和get方法。这两个方法都需要是`synchronized`保护起来的方法。两个方法首先判断列表的大小情况，如果边界调用`wait()`在**当前实例上挂起当前执行的线程并且释放锁资源**。否则的话，从列表存入或者取出一个日期，并且执行`notify()`方法，**通知在这个实例上挂起的全部线程唤醒**。


## 为什么这几个方法在Object方法里，而sleep在Thread里
因为这几个操作是锁的操作，都是绑定于对象的，所有的对象都是可以作为锁对象。

而`sleep`是`Thread`级别的，并不释放锁。`Thread`类不适合作为锁。

## sleep方法详解
`sleep`让线程进入`waiting`状态，并且不占用CPU资源。`sleep`方法在时间未到时候**不会释放锁**的，但是`wait`是释放锁的。sleep方法是可以响应中断的，抛出中断并清除中断状态。谁调用，谁睡眠。
## wait/notify、sleep的异同
相同：都是可以让线程进入阻塞状态；可以相应中断

不同：`wait/notify`是Object类的实例方法，`sleep`是Thread类的静态方法；`wait`需要在同步方法中进行执行，`sleep`不需要；释放锁的区别；sleep必须传参。

* sleep() 和 wait() 有什么区别？

1. sleep() 是 Thread 类的静态方法；wait() 是Object类的成员方法
2. sleep() 方法可以在任何地方使用，**使当前执行的线程休眠**；wait() 方法则只能在同步方法或同步代码块中使用，否则抛出异常Exception in thread "Thread-0" java.lang.IllegalMonitorStateException
3. sleep() 会休眠当前线程指定时间，**释放 CPU 资源，不释放对象锁**，休眠时间到自动苏醒继续执行；wait() 方法**放弃持有的对象锁，进入等待队列，当该对象被调用 notify() / notifyAll() 方法后才有机会竞争获取对象锁，进入运行状态**
4. JDK1.8 sleep() wait() 均需要捕获 InterruptedException 异常

## join方法
>Waits for this thread to die.

**`join()`也是会释放锁的，并且是`Thread`类的实例方法**

当等待其他线程join期间，主线程处于waiting状态。**本质上还是调用了`wait()`，最后Thread方法实现notify。**
```java
// 相当于自旋 isAlive是判断一个线程还没有执行完毕
while (isAlive()) {
	wait(0);
}
```

## yield方法
**这是`Thread`类的静态方法，不释放锁**。在释放以后，只是暂停这个线程，释放锁；然后全部的线程再次去竞争这把锁。*这个方法不是很推荐使用，过度依赖cpu*

作用：释放CPU时间片，线程状态依然是Runnable。与sleep的区别，sleep方法不是随时可以被调度的，yield可以被随时调度。
## park方法
CAS方法中替换state失败了，放入队列中，调用LockSupport.park()让出CPU。这个方法是以线程为单位进行阻塞和唤醒。而wait和notify都是随机的或者唤醒全部

## 其他
* 获取当前线程引用，Thread.currentThread()方法
* start和run方法
* stop（强制停止，不安全），suspend（暂停一个线程，容易死锁），resume（唤醒挂起的线程，容易发生死锁）方法被弃用 



# 线程池
* 为什么要使用线程池？
  
1. 提高相应时间：任务到达时候可以立即使用线程。
2. 降低资源消耗：减小新创建和回收线程的压力，线程数实现线程的复用。
3. 便于系统管理：随意创建线程不利于项目管理。

## 线程池的创建
### 构造参数
标注构造方法，采用`ThreadPoolExecutor`进行构造。
`ExecutorService executorService = new ThreadPoolExecutor(10, 10, 0, TimeUnit.SECONDS, new LinkedBlockingQueue<>());`

原因：，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201025214717665.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70#pic_center)

* corePoolSize 与 maxPoolSize: 前者是只要创建到了就不会减小，即使没有任务，但是maxPoolSize是在队列满了之后，可以灵活的增加新的线程，这个是线程池的上线。

* keepAliveTime: 新增的非核心线程的最大存活时间，超过这个时间没有执行任务就会被回收。

* workQueue: 1.直接队列 SynchronousQueue. 2. 无界 LinkedBlockingQueue 3. 有界 ArrayBlockingQueue

* threadFactory：一般就是用默认的就可以Executors.defaultThreadFactory()

* Handler：拒绝策略，如果超过了最大承载能力，系统如何处理到达的请求。


### 线程的添加规则

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201025214944798.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70#pic_center)

### 线程池的调用逻辑
核心是`ThreadPoolExecutor`类，**这里源码值得一看**涉及到一些函数的可以自己定义自己合适的线程池，如`beforeExcutor`等。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201026195236455.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70#pic_center)

```java
// 线程池调度的核心代码 在ThreadPoolExecutor类中的execute方法
    public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        int c = ctl.get();     // 记录了线程池的状态和线程数
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true)) // 执行的任务和，并且true表示不要超过核心线程的数量
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) { // 检查是否线程池运行，因为前面判断了核心已经满了，这里把任务放入队列中
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0) // 防止没有线程正常运行
                addWorker(null, false);
        }
        else if (!addWorker(command, false)) // 线程满了，且队列满了，无法add进入循环
            reject(command);
    }

```

### 常用默认线程池
*阿里p3c规范不建议这样*

手动创建可以避免一些风险，并且需要和业务匹配。
以下是自动创建的线程池。
1.  `ExecutorService threadpool = Executors.nexFixedThreadPool(4)` 固定数量 
2.  `ExecutorService threadpool = Executors.nexSingleThreadPool()` 单一
以上两个都是`LinkedBlockingQueue`
3. `ExecutorService threadpool = Executors.nexCachedThreadPool()` 可无限创建，core = 0 `SynchronousQueue`
4. `ExecutorService threadpool = Executors.nexScheduledThreadPool(10)`  可以延迟运行 `DelayedWorkQueue`

## 线程数量的设计
CPU密集：线程数量为CPU的1-2倍
IO型：可以大于很多倍。线程数 = cpu*（1+等待时间/工作时间）

>如何判断是 CPU 密集任务还是 IO 密集任务？

CPU 密集型简单理解就是利用 CPU 计算能力的任务比如你在内存中对大量数据进行排序。

涉及到网络读取，文件读取这类都是 IO 密集型，这类任务的特点是 CPU 计算耗费时间相比于等待 IO 操作完成的时间来说很少，大部分时间都花在了等待 IO 操作完成上。


并不线程越多越好。线程之间的切换时需要成本的。内核线程的调度成本主要来自于用户态和和心态的状态切换，主要是状态切换时候的响应中断，保护和恢复现场的成本。
## 停止线程池的方法
* 实例方法`shutdown()`：提交上去关闭的请求，但是主动权在线程池。更安全。可以用实例方法`isShutdown()`判断。或者`isTerminated()`判断, `awaitTerminated()`延迟判断是不是结束了线程池。

* 实例方法`shutdownNow`: 立刻结束。中断所有正在执行的线程，并且返回没有进行的任务。**这个方法并不安全**

## 拒绝任务的策略
* 拒绝时机：线程关闭后或者最大线程和工作队列容量已经饱和。

默认的四种拒绝策略：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201026195153584.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70#pic_center)

具体采用什么样的策略还需要具体业务分析。



## 线程池的组成与实现原理
>线程池的组成
线程池包括：管理器（创建和停止），工作线程，任务队列（BlockingQueue），任务接口（Task）

> Executor家族
线程池继承与实现的关系：
![继承与实现的关系](https://img-blog.csdnimg.cn/20200802203441735.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)


辨析：
* **Executor：是一个顶层接口，只有一个方法`execute()`。执行任务的。**
* **ExecutorServices：继承了Executor的接口，可以管理线程池,`shutdown()`等。**
* **Executors：这是一个工具类。** 可以按照默认的配置创建一些线程池。

>线程池如何复用

每一个线程池拿到任务以后不断检测有没有新的任务，然后执行`run`。


> 使用线程池的注意事项：
1. 防止任务堆积
2. 避免线程过度增加
3. 排查线程泄露：线程执行完毕，无法被回收。

# 并发工具类
不得不提的并发工具类！
![](https://img-blog.csdnimg.cn/20190806221439584.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMjUzMTQ3,size_16,color_FFFFFF,t_70)


> 什么是控制并发流程?

控制线程执行的顺序，实现线程之间的相互配合。

## 常见的并发控制工具类
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201106193643691.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70#pic_center)

## CountDownLatch
倒数门闩，达到一定的数量才可以执行。在倒数结束之前，线程都在等待。

* 构造方法：` CountDownLatch countDownLatch = new CountDownLatch(5);`参数表示几个倒计时量。
* 实例方法：
	1. `.await()`:调用这个方法的线程会被挂起，直到count值为0。可以在主线程使用，等待子线程完成。
	2. `.countDown()`：将count值减1，直到为0，等待的线程被唤醒。
* 典型用法：
	1. **一个线程等待多个线程都执行完毕，在继续自己的工作。** 如，主线程等子线程都完成预处理。
	2. **多个线程等待一个线程的信号，同时开始执行。** 如，模拟压测，同时给服务器压力。只有一个信号，提前初始完毕全部的子线程，利用`await()`方法全部等待，直到主线程发令。
* 注意点: 这个类是**不可以重用**的，无法重置。
## Semaphore
信号量用来限制和管理数量有限的资源的使用。证书的数量有限，类似于令牌。可以现实流量。
* 构造：`Semaphore semaphore = new Semaphore(3, true);`
* 使用流程
 	1. 初始化信号量，给定指定数量的信号量，是否公平策略，一般设置为公平。
 	2. 在需要被限制的代码前调用实例方法`acquire()`或者`acquireUniterruptibly()`或者`tryAcquire()`会返回一个布尔值 。看看有无信号量，如果获取不到，线程会等待。(`acquire()`方法没有参数时默认一个，但是也可以传入参数实现多个信号量的要求。)
 	3. 调用完成之后，调用实例方法`release()`方法释放。**一定要归还**

* 信号量不一定要线程A获取，线程A释放

## Condition
多了一些线程阻塞的条件，原来的wait()只有一个条件，现在可以设计多个。
* 构造: 这个略有不同。

```java
        ReentrantLock lock = new ReentrantLock();
        Condition condition = lock.newCondition();
```

* 实例方法：`condition.await()`线程被阻塞，`condition.signalAll()`唤醒全部正在等待的线程，`signal()`具有公平性，唤起等待时间最长的那个线程。

* 编写生产者消费者模式：

```java
import java.util.PriorityQueue;
import java.util.concurrent.ThreadLocalRandom;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Demo{
    private int queueSize = 10;
    private PriorityQueue<Integer> queue = new PriorityQueue<Integer>(queueSize);
    private Lock lock = new ReentrantLock();
    private Condition notFull = lock.newCondition();
    private Condition notEmpty = lock.newCondition();

    public static void main(String[] args) {
        Demo demo = new Demo();
        Concumer concumer = demo.new Concumer ();
        Producer producer = consumerdemo.new Producer();
        Thread t1 = new Thread(concumer);
        Thread t2 = new Thread(producer);
        t1.start();
        t2.start();

    }

    class Concumer implements Runnable{

        @Override
        public void run() {
            try {
                consume();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        private  void consume() throws InterruptedException {
            while(true){
                lock.lock();
                try {
                    while(queue.size() == 0){
                        System.out.println("empty");
                        try {
                            notEmpty.await();
                        }catch (InterruptedException e){
                            e.printStackTrace();
                        }
                    }
                    queue.poll();
                    notFull.signalAll();
                    System.out.println("get and leave"+queue.size());
                }finally {
                    lock.unlock();
                }
            }
        }
    }

    class Producer implements Runnable{

        @Override
        public void run() {
            try {
                preduce();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        private  void preduce() throws InterruptedException {
            while(true){
                lock.lock();
                try {
                    while(queue.size() == queueSize){
                        System.out.println("full");
                        try {
                            notFull.await();
                        }catch (InterruptedException e){
                            e.printStackTrace();
                        }
                    }
                    queue.offer(1);
                    notEmpty.signalAll();
                    System.out.println("make and leave"+queue.size());
                }finally {
                    lock.unlock();
                }
            }
        }
    }
}

```

## CyclicBarrier
将大量线程相互配合，汇合之后再继续执行，
* 构造：传入的参数是需要等待的对象和Runnable接口，可以在完成集和之后进行一些操作。
```java
        CyclicBarrier cyclicBarrier = new CyclicBarrier(5, new Runnable() {
            @Override
            public void run() {
                System.out.println("集合");
            }
        });
```

* 实例方法：`cyclicBarrier.await()`方法让先完成的线程等待。只要等待的个数达到规定的数量，就继续执行。需要注意的话，这个方法是可以重用的，也就是只要等待满足5个就可以继续执行。
* 与countdownlatch的不同：**本工具针对线程，count针对的是事件，只要是某个事件发生了就可以。** 并且本工具有功能。
  


# 常用的并发容器
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020110419324327.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70#pic_center)

重点的主要是`ConcurrentHashMap` , `CopyOnWriteArrayList`， 和接口`BlockingQueue`

## 淘汰的并发容器
* `Vector`：性能不好，主要是全部都是`synchronized`保护起来的方法，并发性不好。
* `Hashtable`: 线程安全的hashmap，同时是性能不好，sychronized修饰。
* `Collections.synchronizedList(new ArrayList<E>())`和`Collections.synchronizedMap(new HashMap<K, V>())`: 把synchronized加在了代码块上。

锁的粒度太大。

## ConcurrentHashMap
我们首先从HashMap的源码开始分析，map在Java里面是一个很大的家族体系。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201104195230250.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70#pic_center)

* 常用的家族：
	`HashMap`: 朴实无华的字典结构
	`LinkedHashMap`: 可以保留插入的顺序，可以按照顺序查找。
	`TreeMap`: 实现了排序接口，可以按照key的顺序进行一定的处理。

* 为什么Hashmap是线程不安全的：
	1. 同时put碰撞导致数据丢失。
	2. 同时扩容导致数据调式
	3. 可能导致死循环。

[源码解析](https://www.cnblogs.com/zerotomax/p/8687425.html#go4)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201114211205559.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70#pic_center)

采用多个segment的思路，每个segment都有一个独立的ReentrantLock锁，彼此互不影响，提高了并发效率。默认是16个的segment，初始化之后不可以扩容。一个Segment实例就是一个小的哈希表

在任何一个构造方法中，都没有对存储Map元素Node的table变量进行初始化。**而是在第一次put操作的时候在进行初始化。**

在1.8中，采用的是类似与HashMap的方法，并且是分为了很多个node，因此每一个节点都具有并发性。并且使用了CAS+synchronized的方法保证并发性。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201114211530889.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70#pic_center)

**核心方法**：
[源码解析](https://www.cnblogs.com/zerotomax/p/8687425.html#go4)
`putVal`这个方法在第一天添加节点时候使用的是cas的策略，在增长链表或者给红黑树加节点这个环节还是用了synchronized。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201104202235540.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70#pic_center)![在这里插入图片描述](https://img-blog.csdnimg.cn/20201104202259569.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70#pic_center)

还可以使用一些线程安全的方法如`map.replace(key, oldval, newval)`这个可以实现类似于CAS的操作，返回一个布尔逻辑值告诉我们操作是否如愿。`map.putIfAbsent(key, val)`如果当前key不存在，就放入，否则取出。

* Q： put方法的解析

     * 当添加一对键值对的时候，首先会去判断保存这些键值对的数组是不是初始化了，如果没有的话就初始化数组。
     * 然后通过计算hash值来确定放在数组的哪个位置。如果这个位置为空则直接添加（CAS），如果不为空的话，则取出这个节点来。如果取出来的节点的hash值是MOVED(-1)的话，则表示当前正在对这个数组进行扩容，复制到新的数组，则当前线程也去帮助复制。
     * 最后一种情况就是，如果这个节点，不为空，也不在扩容，则通过**synchronized**来加锁，进行添加操作，然后判断当前取出的节点位置存放的是链表还是树；如果是链表的话，则遍历整个链表，直到取出来的节点的key来个要放的key进行比较，如果key相等，并且key的hash值也相等的话，则说明是同一个key，则覆盖掉value，否则的话则添加到链表的末尾；如果是树的话，则把这个元素添加到树中去。
     *  最后在添加完成之后，会判断在该节点处共有多少个节点（注意是添加前的个数），如果达到8个以上了的话则调用treeifyBin方法来尝试将处的链表转为树，或者扩容数组

## 并发队列

* 为什么使用：可以在线程之间传递数据；不用操心安全性
* 线程安全的队列：
* ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201104222517128.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70#pic_center)

## 阻塞队列BlockingQueue
* put，take：会阻塞
* add，remove，element：如果无法实现，会返回异常
* offer, poll, peekL：如果失败会返回布尔值。

**源码分析：LinkedBlockingQueue分别使用了putLock和takeLock，利用signal进行通信。这样进一步提高了效率。**

## 非阻塞队列ConcurrentLinkedQueue
采用了CAS的方法实现并发的线程安全。


## CopyOnWriteArrayList
* 构造：`CopyOnWriteArrayList<Integer> list = new CopyOnWriteArrayList<>(new Integer[]{1, 2, 3});`
* 适用场景：读操作尽可能的快，即使写慢一些也无所谓。读多写少。

* 读写规则：**只有写写之间存在互斥。**实际上是修改了备份。因此存在一个小的问题，在迭代中修改的东西没法立刻体现。**迭代器能的得到的数据，取决于创建的时间，而不是迭代的时间。**
* 原理：创建了新的副本，读写分离的思想。
* 缺点：数据一致性存在问题；内存占用问题。

**[源码分析](https://www.cnblogs.com/developer_chan/p/11490517.html)**
使用了`Reentrantlock`进行add方法



# 并发中的锁
>锁的分类

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201028204508298.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70#pic_center)

## synchronized
Java的并发编程老祖先，1.6之前性能比较差。后来被优化了，现在和ReentrantLock大差不差了。大量源码还是与使用了Synchronized。

> 使用方法
1. 修饰实例方法：如果希望进入代码块需要取得当前对象的实例锁
2. 修饰静态方法：对于类加锁了，使用的class锁，只有一个线程可以使用类的静态方法。但是实例方法还是可以访问的。
3. 修饰代码块：`synchronized(this)` 对于特定的对象加锁。

> 双重检验实现对象单例
```
public class Singleton {
    // 考点1：需要volatile修饰，保证可见性和禁止重排序。
    private volatile static Singleton uniqueInstance;

    private Singleton() {
    }

    public  static Singleton getUniqueInstance() {
       //先判断对象是否已经实例过，没有实例化过才进入加锁代码
        if (uniqueInstance == null) {
            //类对象加锁
            synchronized (Singleton.class) {
                // 二次检验，可能这个时候因为多线程的原因发生了修改。
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```
* 为什么需要volatile关键字？
因为new对象这个操作并不是一个原子性的操作，可以分为三步：为对象分配内存空间，初始化对象，将对象指向分配的内存地址。这个过程可能发生重排序，比如还没初始化但是先指向地址了，这样就会发生错误。

~~个人认为还考虑了可见性，比如在检测的时候需要保证实例是从主内存中flush到工作内存中的最新数据。~~

* synchronized的底层原理是什么？
  
>代码块同步: 通过使用monitor enter和monitor exit指令实现的。试图获取和放开monitor。

>同步方法: 在flag位置用ACC_SYNCHRONIZED修饰。表示了方法是同步的。

都是希望获取对象监视器monitor的获取。多线程下synchronized的加锁就是对同一个对象的**对象头**中的MarkWord中的变量进行CAS操作。这样别的线程过来就可以从对象头得到锁的情况。

早期比较重量级：因为监视器锁（monitor）是依赖于底层的操作系统的 Mutex Lock 来实现的，Java 的线程是映射到操作系统的原生线程之上的。**如果要挂起或者唤醒一个线程，都需要操作系统帮忙完成，而操作系统实现线程之间的切换时需要从用户态转换到内核态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高。**


* jdk6对synchronized的优化有哪些？
jvm做了一些优化。锁可以升级, 但不能降级. 即: 无锁 -> 偏向锁 -> 轻量级锁 -> 重量级锁。
偏向锁比较好说，对象头存储了一个线程Id。下次还是这个id直接使用，别的线程来了会查看并且CAS竞争以下。此时会膨胀为轻量锁，尝试CAS的方法将对象头中的markdown指向自己的线程。如果失败了会自旋一会，如果自旋也失败了。直接标记为强竞争，升级为重量锁，后续线程来了直接排队。

偏向锁适合一个线程对一个锁的多次获取的情况; 轻量级锁适合锁执行体比较简单(即减少锁粒度或时间), 自旋一会儿就可以成功获取锁的情况。

* synchronized和ReentrantLock区别
前者依靠JVM，后者API。后者等待可以被中断，可以实现公平锁，可以有多个条件。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210308143344477.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)

## Lock接口
Lock和Synchronized是两个最常见的锁。
Lock接口最常见的实现类是`ReentrantLock`。
## Reentrantlock源码解析
[参考文献](https://www.cnblogs.com/gocode/p/SourceCodeAnalysisOfReentrantLock.html)

ReentrantLock是一个支持重入的排他锁，即同一个线程中可以多次获得锁。ReentrantLock主要利用CAS+AQS队列来实现。[AQS](#aqs)

ReentrantLock类中有Sync、FairSync、NofairSync这三个静态内部类。Sync是一个继承于AQS的一个抽象类。FairSync 和NofairSync都继承自Sync，分别表示公平锁、非公平锁。它们两者的类定义差别很小，只有尝试获取锁的方法不同，FairSync使用自已定义的tryAcquire，而NotFairSync使用 给父类Sync的nonfairTryAcquire（int)方法实现，而两者尝试释放锁的方法都是继承父类Sync的tryRelease(int）。默认是非公平锁，需要竞争的。

当获取锁时候，先通过CAS尝试获取锁。如果此时已经有线程占据了锁且不是自己，那就加入AQS队列并且被挂起。当锁被释放之后，排在CLH队列队首的线程会被唤醒，然后CAS再次尝试获取锁。在这个时候，如果是非公平锁：如果同时还有另一个线程进来尝试获取，那么有可能会让这个线程抢先获取；如果是公平锁：如果同时还有另一个线程进来尝试获取，当它发现自己不是在队首的话，就会排到队尾，由队首的线程获取到锁。


非公平锁的流程：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021030516323988.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)


> FairSync

公平锁和非公平锁不同之处在于，公平锁在获取锁的时候，不会先去检查state状态，而是直接执行aqcuire(1
### 构造
ReentrantLock有一个Sync类型的成员变量sync，这个成员变量在构造方法中被实例化。
### 可重入机制

如果没有任何线程获取到此锁，尝试CAS尝试更新state，若此CAS更新成功,则成功获取锁并将当前线程设置为锁的占有线程，若CAS更新抢购，则获取锁失败。若当前线程获取之前已经获取到此锁，则将重复获取到锁的次数state自增。

FairSync使用自已定义的tryAcquire(int)，而NotFairSync将tryAcquire（int)委托给父类Sync的**nonfairTryAcquire**（int)方法实现，而两者尝试释放锁的方法都是继承父类Sync的**tryRelease**(int）。

一线程获取了n次锁，那么也需要经过n次释放，锁才能完成释放，其他线程才能获取到此锁。代码实现思路：每次释放锁让AQS的state属性自减，当state为0时，表明锁被完全释放了。

 **Synchronized缺点**：
 
* 效率低：锁的释放困难，无法灵活释放锁。(1.8版本的优化以后效率大幅度提高)
* 无法知道是否成功获取锁。

还有一个更为神奇的ReentrantReadWriteLock，非公平实现时候。写锁可以插队，读锁只有在头节点不是写锁的时候插队（防止饥饿，并且提高效率）。

### Lock接口中的方法
* `lock()`: 最普通的方法，不会在异常时自动释放锁。因此最佳实践是，**在finally中释放锁， 保证发生异常时锁可以被释放**。

缺点： 无法被中断，一旦陷入死锁，lock()就会进入永久等待。

* `tryLock()`: 尝试获取锁，返回boolean。这个方法是可以立刻返回的。
* `tryLock(long time, TimeUnit )`： 等待一定时间，然后判断。
* `lockInterruptibly()`: 等待时间无线，等待锁的过程中，线程可以被中断。
* `unlock()`: 一定要放在finally()



## 乐观锁与悲观锁
* 互斥锁的缺点：
    阻塞带来的性能缺点。
    死锁缺陷。**乐观锁是免疫死锁的。**

* 乐观锁（CAS）：认为发生冲突是小概率的。操作时候不锁住对象，在更新时候进行检查，发生了冲突在回滚修复。典型例子：原子类，并发容器。

> 适用场合：

悲观锁： 1. 临界区有IO操作；2. 临界区代码复杂； 3. 临界区竞争激烈

乐观锁：适合并发写入少，大部分是读取的场合。

## 可重入锁与非可重入锁

* 可重入：可以一个线程请求多个锁。可以在一定程度上减小死锁。取得了某把锁的线程可以继续获得这个锁。在拿到锁之后继续可以拿到其他的锁。

* 源码实现：AQS框架实现的。
典型例子:`ReentrantLock()`

## 公平与非公平
* 公平锁就是按照线程请求的顺序来分配锁；非公平锁在需要的场合可以插队。

优点：一定程度上，非公平锁是可以提高效率，避免唤醒带来的空档期。

缺点：可能带来饥饿，一些线程永远等不到锁。

公平锁：`new ReetrantLock(true)`

## 共享锁与排他锁
一个锁是否只能一个线程持有。典型的共享锁就是`ReentrantReadWriteLock`，对于读是完全没有干扰的，但是写是封闭的。

```java
    ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock();
    // 读锁和写锁都是可以单独的加锁解锁的
    ReentrantReadWriteLock.WriteLock writeLock = reentrantReadWriteLock.writeLock();
    ReentrantReadWriteLock.ReadLock readLock = reentrantReadWriteLock.readLock();
```
### 读写锁
并发包中对ReadWriteLock接口的实现类是ReentrantReadWriteLock，这个实现类具有下面三个特点。

1. 具有与ReentrantLock类似的公平锁和非公平锁的实现：默认的支持非公平锁，对于二者而言，非公平锁的吞吐量由于公平锁；
2. 支持重入：读线程获取读锁之后能够再次获取读锁，写线程获取写锁之后能再次获取写锁，也可以获取读锁。
3. 锁能降级：遵循获取写锁、获取读锁在释放写锁的顺序，即写锁能够降级为读锁。锁降级的概念：如果当先线程是写锁的持有者，并保持获得写锁的状态，同时又获取到读锁，然后释放写锁的过程。
* 读写锁的交互方法：默认不许插队，但是可以锁的降级。

读写锁是分开的两个锁，writelock和readLock。它的自定义同步器（继承**AQS**）需要在同步状态（一个整型变量state）上维护多个读线程和一个写线程的状态，使得该状态的设计成为读写锁实现的关键。如果在一个整型变量上维护多种状态，就一定需要“按位切割使用”这个变量，**读写锁将变量切分成了两个部分，高16位表示读，低16位表示写。**


**公平锁**：完全不许插队

**非公平锁**：**写锁可以随时插队，读锁只在等待队头不是写锁时候插队。**

**降级**：从写锁变为读锁。但是不能反过来。在`writelock.lock()`情况下可以请求`readlock.lock()`。升级会容易造成死锁，使用写锁的前提是不能有读锁，因此可能导致死锁（两个同时升级）。

## 自旋锁和阻塞锁
线程的状态转换消耗的时间比用户执行代码需要的视角还多。这时对线程执行自旋，完成自旋以后再判断是不是可以获取锁。

缺点：如果前面的线程一直占着锁，其实是浪费cpu资源。

atomic包下的类基本都是自旋锁实现的，AtomicInteger的实现，自旋锁的原理是CAS，再调用unsafe自增是，就是一直while死循环，直到修改成功。

* 适用场合：多核，并发度低，临界区简单。
## 中断锁与不可中断
典型就是`synchronized`和`reentrantlock`

## 锁优化（JVM帮助实现）
* 偏向锁：如果一个线程取得了锁，锁进入偏向模式。当这个线程再次请求时候，无需任何同步操作。（在竞争激烈的场景下失效）
* 轻量级锁：在偏向锁失败以后，虚拟机会将对象头部作为指针 指向持有锁的线程堆栈内部，判断一个线程是否持有对象锁。 如果失败会膨胀为重量级锁。
* 自旋锁和自适应：循环了几次之后自己放弃自旋锁的方法。
* 锁消除：对于没有必要使用的锁不使用
* 锁粗化：合并几个锁。

[锁升级](https://www.cnblogs.com/mingyueyy/p/13054296.html)
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210315095207920.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)


锁的信息都是存储在对象头中
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210315095423469.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)

# CAS原理
实现不能被打断的并发操作，Compare and Swap。CAS有三个操作数：内存值V，预期值A，要修改的值B。**当且仅当预期值与内存值一样，才将内存值进行修改**。每次有一个线程可以成功执行。**通过硬件指令保证了原子性。**



本质实现：使用CPU保证原子性。
## 应用场景：
* 乐观锁，采用cas的方法实现并发
* 并发容器：concurrentHashMap
* 原子类：AtomicInteger，**这个类加载`Unsafe`工具，可以直接操作内存数据。并且volatile保证可见性。Unsafe是native方法。**

## 缺点
* ABA问题：5改成7，7又改成5。会错误的认为没有发生修改。
* 解决方法：在使用的时候加上时间标签，`AtomicStampedReference`,除了对比数值还需要对比时间戳。
* 自旋时间比较长：高并发的场景下会消耗资源

# ThreadLocal类
* ThreadLocal的使用场景
1. 工具类不是线程安全的，我们让每个线程都有一个独立的工具类。
2. 避免参数传递的麻烦。
![1. 1. 1.](https://img-blog.csdnimg.cn/20200802210701629.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)


> 场景1.每个线程需要一个独享的对象，常用在工具类中
*  当多个线程多次使用同一个线程不安全的对象时，会发生线程安全问题。可以使用线程池+synchronized的方法，但这种方法的性能差。

* 采用ThreadLocal方法。 **创建一个安全类`SafeThreadLocal`，在这类中新建一个静态的`ThreadLocal<> = new ThreadLocal<>();`对象。** 这样所有的线程都可以调用。 **这个对象需要重写`initialValue()`方法，也就是每个ThreadLocal都可以初始化生成一个属于线程的`dataformat`实例**。对10个线程都创建一个dateFormat对象。每一个dateFormat的是线程安全的，**线程在调用时，使用`类.实例名.get()`得到属于每类自己的`dataformat`实例。**


在jdk1.8中，有了更加优雅的方法，可以函数式编程的方法使用`withInitial()`，去生成新的副本。




>场景2. 每个线程内保存一个内部变量，不同的方法可以直接使用，常用于对一个对象进入多个方法。
比如多个用户请求，每个线程都对应了不同的用户信息，我们希望在一个线程内维护一个变量，在调用方法的时候是通用的。可能的做法包括采用ConucurrentHashMap等。但是这种方法会导致性能降低。

* 采用`ThreadLocal`可以不影响性能，且无需层层传递参数。在线程周期内，通过**静态**的`ThreadLocal`实例的`get()`方法取得自己`set()`的对象，避免了传参的麻烦。实现统一个线程内不同方法之间的共享。

* **在定义类中定义静态的`ThreadLocal`，不需要重写`initialValue()`方法**。**在线程中手动调用`类名.静态实例名.set()`**，在同一个线程的其他程序调用时，**直接`类名.静态实例名.get()`.**、


```java

public class ThreadLocalExample implements Runnable{

     // SimpleDateFormat 不是线程安全的，所以每个线程都要有自己独立的副本
    private static final ThreadLocal<SimpleDateFormat> formatter = ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyyMMdd HHmm"));

    public static void main(String[] args) throws InterruptedException {
        ThreadLocalExample obj = new ThreadLocalExample();
        for(int i=0 ; i<10; i++){
            Thread t = new Thread(obj, ""+i);
            Thread.sleep(new Random().nextInt(1000));
            t.start();
        }
    }

    @Override
    public void run() {
        System.out.println("Thread Name= "+Thread.currentThread().getName()+" default Formatter = "+formatter.get().toPattern());
        try {
            Thread.sleep(new Random().nextInt(1000));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //formatter pattern is changed here by thread, but it won't reflect to other threads
        formatter.set(new SimpleDateFormat());

        System.out.println("Thread Name= "+Thread.currentThread().getName()+" formatter = "+formatter.get().toPattern());
    }

}
```


* ThreadLocal的优点
1. 可以实现线程安全。
2. 不需要加锁，提高了实行效率。
3. 更高效的利用内存，节省开销。只需要建立与线程池中线程数量相同的实例，节省内存。
4. 避免的传参的繁琐。在同一个线程中可以共享变量。

> 源码分析

`ThreadLocalMap`是一个写在**线程**里面的关键对象，保存了一个字典。

在`set`和`get`时，我们首先获得当前线程的对象，然后`getMap`方法得到线程的`ThreadLocalMap`，这个Map里面保存了这个线程的全部local。然后进行类似与Hashmap的操作。**这个ThreadLocalMap是写在Thread类内部的成员。**

```java
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
```

**简单来说就是每一个`Thread`都对应了一个`ThreadLocalMap`，而每一个`ThreadLocalMap`里面可以存在多个`ThreadLocal`。这样是为了在一个线程内可以读取多个变量。也就是说，我们可以新建很多个ThreadLocal。**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200802231810408.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)

> 重要方法解析
1. `initialValue()`: 返回线程对应的初始值，**延迟加载，只有在调用get时候才会触发**。如果之前`set`了值，就不需要了。**每个线程调用最多一次就可以**。如果不重写，就会返回null。
2. `set(T t)`: 为这个线程设置一个新值
3. `get()`: 得到这个线程对应的value。
4. `remove()`: 删除线程的值。清空**当前线程**的ThreadLocal。

> ThreadLocalMap类
也就是Thread.threadLocals。核心是一个键值对数组`Entry[] table`

* 处理哈希冲突：冲突是开放寻址法。
* 传统哈希：初始时拉链法，后续变为红黑树。

> ThreadLocal注意点
* 内存泄露：某个对象不在使用，但占用的内存无法被回收。
1. key 是一个**弱引用**，是可以被垃圾回收的。
2. value是一个**强引用**。如果线程结束，线程是被回收的。但是如果使用线程池，线程保持，因此value无法被回收，引发内存泄漏。JDK已经考虑了，在调用`set`，`remove`，`rehash`方法时候会把key是null的的value也设置为null。但是一旦ThreadLocal不用了，一般也很难去调用remove，容易造成泄露。
3. 避免方法：**使用完ThreadLocal，主动调用remove。也就是说，在最后一个方法调用完ThreadLocal，需要主动remove，避免内存泄漏。**

* 共享对象:
注意不要存入一个static对象，否则还是会导致线程安全的问题。




# 原子类
**原子特性**：一组操作时不可以被打断的，要不没进行，要不完整的完成。即使在多线程的情况下。

**特点**：粒度更细，更轻量。但是在高度竞争情况下效率降低。


![在这里插入图片描述](https://img-blog.csdnimg.cn/20201030204446835.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70#pic_center)


 
## AtomicInteger
 生产实例`AtomicInteger atomicInteger = new AtomicInteger();`
 
 实例具有的方法：
* `get()`: 获取当前值
* `getAndSet(int val)`: 获取当前值，并且设置新的值
* `getAndIncrement()`： 获取当前值，并且自增。A++
* `getAndDecrement()`: 获取当前值，并且自减。A--
* `getAndAdd(int delta)`：可以加很多
* `compareAndSet(int expect, int update)`： CAS的操作


## AtomicIntegerArray 
 生产实例`AtomicIntegerArray atomicIntegerArray = new AtomicIntegerArray (1000);`生成一个数组，这个数组中的数字都是整形，可以被原子操作更新的。
 
 实例具有的方法：
* `get(index)`: 获取当前值
* `getAndSet(index，int val)`: 获取当前值，并且设置新的值
* `getAndIncrement(index)`： 获取当前值，并且自增。A++
* `getAndDecrement(index)`: 获取当前值，并且自减。A--
* `getAndAdd(index, int delta)`：可以加很多
* `compareAndSet(index, int expect, int update)`： CAS的操作

## AtomicReference
让一个对象保持原子性。主要方法就是，`compareAndSet(expect, current)`。

生成实例`AtomicReference<?> name = new AtomicReference<>();`

## AtomicIntegerFiledUpdate
把普通变量升级为原子类变量。

* 构造方法：

```java
public class test {
    static Candidate tom;
    public static AtomicIntegerFieldUpdater<Candidate> socreUpdate = AtomicIntegerFieldUpdater
            .newUpdater(Candidate.class, "score");
    public static class Candidate{
        volatile int score;
    }
    // 之后再使用的时候，使用socreUpdate.getAndIncrement(tom);
}
```

* 方法原理：
实际上它是反射实现的，因此**必须是public的变量才可以被修饰升级**。并且不支持static方法，同样因为是反射。

## LongAdder
是Java 8引用的，再高并发的环境下，**LongAdd比AtomicLong的效率高**，本质是空间换时间。

构建`LongAdder counter = new LongAdd();`，实例方法：`.increment()`。这个操作的自加比AtomicLong快。

* 原理：AtomicLong每一次都要进行flush和refresh。 LongAdder不需要每每次都flush，而是在每个线程的内部都有一个自己的计数器，不会和其他线程的计数器冲突。内部实际上有一个`base`变量当不激烈时候，直接累加，否则使用`Cell[]`数组，各个线程分散累加到自己的槽里，最后在累加。

* 使用场合：LongAdder使用高并发下的统计求和，方法单一；AtomicLong还有其他的方法。




# final不变性
对一个对象而言，被创建以后，状态不再修改，就是不可变的。这种对象是线程安全的，因为最多只能并发读取，但是不能修改。

> final关键字
* 作用：**类防止被继承；方法防止被重写，变量防止被修改。**

当`final`修饰一个对象时候，只是**保证了对象的引用地址不会发生改变，但是对象引用的内容还是可能被修改的。**

>1. 修饰变量


因为final的赋值只能进行一次，因为需要准寻赋值时机。
* final 对于类变量
final必须被初始化赋值。1.直接定义时候，2.构造函数中`this.a = a`, 3.初始化代码块中。
* final静态变量
1、直接在等号右边赋值，2.static初始代码块赋值。
* 方法中的final
在使用前进行赋值即可。

>2. 修饰方法
* 不能修饰构造方法
* 修饰其他方法，防止被override。**静态方法也是不能被重写，但是可以写一个一模一样的静态方法（JVM的静态绑定）**

>3.  修饰类
典型的就是`String`不可以被继承。

## 不变性与final的关系
* 对于基本数据类，被final修改具有不变性
* 对于对象类型，需要该对象保证自身被创建以后，状态才会永久不可变。

注意：并不是把所有的属性都声明为final就是不可变的，可能存在对象的类型。但是代码中可以保证这个对象不会被操作（对象方法封闭发布没有发生溢出），从而实现不可变。

> 栈空间
> 
把变量写在线程的内部，线程调用方法，方法内部定义的`int`之类的是不线程共享，因此是可以保证线程安全的。


# Runnable与Callable接口
## Runnable的缺陷
* 不支持返回一个返回值，run方法被void的修饰。`public abstract void run();`
* 无法抛出check异常，也是因为原始接口的run方法是没有抛出异常的修饰的。只能用try，catch。

## Callable 接口
类似与Runnable，被其他线程执行的任务。需要重写`call`方法，并且是有返回值的。而且可以抛出异常。` V call() throws Exception;`
## Future类
* 作用：用子线程去执行，主线程不用去等待执行结束。只需要在合适的时刻去调用`get`方法获取结果就可以。

任务提交给线程池时候需要使用`submit()`而不是`excute()`，后者不返回东西，而前者会返回一个Future对象。
## Callable和Future的关系
* 可以用`Future.get()`来获取Callable接口返回的执行任务，
* `Futuren.isDone()`来判断任务是否已经执行完成或者取消任务。
* 如果`call()`方法还没执行完，调用`get()`方法的线程会被阻塞，直到`call()`方法返回结果，主线程才会恢复。
* 可以认为Future是一个存储器，存储了call()这个任务的结果。

## 重要方法
* `get()`：任务正常完成会返回，如果没有完成会被线程会被阻塞。如果call方法抛出异常，get方法抛出的异常都会是`ExecutionException`，并且这异常是在get方法请求的时候才会被抛出。如果任务被取消，get方法抛出`CancellationException`。任务超时的话抛出`TimeOutException`。可以带超时的时间参数。
* `cancel()`:  提前结束提交的任务。可以传入一个Boolean参数，决定要不要利用interrupt进行打断当前正在执行的任务。如果任务还没开始执行或者正在执行，会返回true；如果已经执行完毕了，返回false。
* `isDone()`：只是执行完毕了，未必执行成功了。
* `isCanceled()`：判断是否被取消了

* 构造方法：`Future<?> f = service.submit(new Task());`，没有完成计算之前，f是空的。然后可以用`f,get()`得到返回值。这里需要注意，task是需要继承`Callable<>`类的，并且重写`call()`方法。或者调用lambda方法。

### 方法一：继承Callable<>类

```java
import java.util.concurrent.*;

public class FutureTask {

    static class Task implements Callable<Integer>{
        @Override
        public Integer call() throws Exception {
            Thread.sleep(2000);
            return 10;
        }
    }
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        ExecutorService service = Executors.newFixedThreadPool(1);
        Future<Integer> future = service.submit(new Task());
        Thread.sleep(3000);
        System.out.println(future.get());
        service.shutdown();
    }
}

```

### 方法二：lambda表达式
```java
import java.util.concurrent.*;

public class FutureTask {

    public static void main(String[] args) throws InterruptedException, ExecutionException {
        ExecutorService service = Executors.newFixedThreadPool(1);
        // 使用lambda表达式
        Callable<Integer> calls = ()->{
            Thread.sleep(2000);
            return 10;
        };
        Future<Integer> future = service.submit(calls);
        Thread.sleep(3000);
        System.out.println(future.get());
        service.shutdown();
    }
}
```
## JDK中的Future

`Callable`是一个接口，内置了`call()`方法，并且是可以抛出异常的。
`Future`是一个接口，内置了`cancel`，`isCancelled()`，`get()`，`isDone()`，`get(long, TimeUnit)`这几个方法。
`RunnableFuture`是一个接口，利用接口的多继承，继承了`Future`、`Runnable`这两个接口，并且有一个`run`方法用于完成真实的运算。
`FuturnTask`是一个实现类，继承了`RunnableFuture`接口。并且执行FT的run方法时，会调用`Callable`接口中的`call()`方法完成计算，并且存储，等到`get()`方法调用时返回存储的结果。
## 用FutureTask创建Future
FutureTask是一种包装器，可以把Callable转化为Future和Runnable，实现了两者的接口。
Fi 
第一步：建立一个Callable任务，` Callable<V> callable = new Callable<V>(){“重写call方法”};`
第二步：生成一个FutureTask类，传入Callable类型，`FutureTask<Integer> ft= new FutureTask<>(callable );`
第三步：线程执行`ft.run`，或者向线程池中提交`excutor.submit(ft)`。
第四步：得到返回值`ft.get()`

具体的实现可以参考高性能缓存。

## Future的注意点
无法再次初始化，生命周期无法后退，只能在创建新的Future去完成新的任务。


# AQS
（AbstractQueueSynchronizer）
其实被广泛的应用，ReentrantLock和Semaphore，CountDownLatch等都继承了AQS类。实现了包括，同步状态的原子性管理，线程的阻塞和接触阻塞，队列的管理等。可以认为是一个构造工具类的工具类。

帮助其他工具类实现了线程挂起，线程排队等具体的操作。

AQS 核心思想是，如果被请求的共享资源空闲，则**将当前请求资源的线程设置为有效的工作线程**，并且将共享资源设置为**锁定状态**。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制 AQS 是用 **CLH 队列锁**实现的，即将暂时获取不到锁的线程加入到队列中。

>CLH(Craig,Landin,and Hagersten)队列是一个虚拟的**双向队列**（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS 是将每条请求共享资源的线程封装成一个 CLH 锁队列的一个结点（Node）来实现锁的分配。

## 三大核心部分之state
state的具体含义在不同的实现类中不同，比如在`Semaphore`中，表示剩余的许可证数量，在`CountDownLatch`中表示还需要倒数的数量。`ReentrantLock`中表示了锁的可重入数量，为0表示当前锁是释放的。

是用`volatile`修饰的，会被并发的修改，因此需要保证线程安全。

## 三大核心部分之FIFO队列
用于存放等待的线程，维护一个等待的线程队列，把所有线程都挂起并放在这个队列里。CLH 锁队列**双向链表的结构**

## 三大核心部分之获取/释放
* 获取方法：获取操作时依赖state变量的， 经常会阻塞。都会用到CAS
* 释放方法：不会阻塞线程，操作state。也会用到CAS

## AQS对资源的共享方式
1. Exclusive（独占）：只有一个线程可以执行。可以分为公平与非公平。
2. share（共享）：可以多个线程一起。

AQS 使用了**模板方法模式**，自定义同步器时需要重写下面几个 AQS 提供的模板方法：

    isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。
    tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
    tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
    tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
    tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。


# 并发存在的问题
## 死锁
* 什么是死锁：
 发生在并发中，两者互不相让，互相持有对方所需要的资源，又不主动释放，导致程序卡死。

* 死锁的影响
死锁的影响在不同的系统中不同的。在数据库中就是可以检测并且放弃事务的。但是JVM无法自动处理死锁。

死锁发生的几率不高但是危害大，压力测试无法找出所有的潜在死锁。
### 银行家算法？
找到一个所有请求的资源都可以满足的，一次性把资源给他，然后再一次性收回。
### 发生死锁的条件
学过操作系统的朋友都知道产生死锁必须具备以下四个条件：

    互斥条件：该资源任意一个时刻只由一个线程占用。
    请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
    不剥夺条件:线程已获得的资源在未使用完之前不能被其他线程强行剥夺，只有自己使用完毕后才释放资源。
    循环等待条件:若干进程之间形成一种头尾相接的循环等待资源关系

图解Java并发设计认为三点

1. 存在多个资源块
2. 线程持有某个资源块时还希望获得其他的资源块
3. 获取资源的顺序并不固定。 

> 死锁的修复
可以采用`ThreadMXBean `这一工具类。

```java
        ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
        long[] deadlockedThreads = threadMXBean.findDeadlockedThreads();
        if (deadlockedThreads != null && deadlockedThreads.length>0){
            for (int i = 0; i < deadlockedThreads.length; i++) {
                ThreadInfo threadInfo = threadMXBean.getThreadInfo(deadlockedThreads[i]);
                System.out.println("find "+threadInfo.getThreadName());
            }
        }
```

### 实际工程中如何避免死锁
* 线上问题都需要防患于未然，无法线上及时的修复
* 首先需要保存现场，保存堆栈的信息，然后重启服务器等。首先保证用户的体验。
* 暂时保证线上代码的安全， 利用保存的堆栈信息等，排查死锁，修改代码。

修复策略：
1. 避免策略：哲学家就餐，设置一致的锁的获取顺序。
2. 检测与恢复策略：每隔一段时间检测是否有死锁，如果有就剥夺某一个资源，打开死锁。
3. 鸵鸟策略：直到发生死锁，再进行修复。

### 以哲学家就餐问题为例
[参考](https://blog.csdn.net/weixin_44801959/article/details/107441403)
```java
public class test {

    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        int sum = 5;
        Chopstick[] chopsticks = new Chopstick[sum];
        for (int i = 0; i < sum; i++) {
            chopsticks[i] = new Chopstick();
        }
        for (int i = 0; i < sum; i++) {
            exec.execute(new Philosopher(chopsticks[i], chopsticks[(i + 1) % sum]));
        }
    }
}

// 筷子
class Chopstick {
    public Chopstick() {
    }
}
class Philosopher implements Runnable {

    private Chopstick left;
    private Chopstick right;

    public Philosopher(Chopstick left, Chopstick right) {
        this.left = left;
        this.right = right;
    }

    @Override
    public void run() {
        try {
            while (true) {
                Thread.sleep(100);//思考一段时间
                System.out.println(Thread.currentThread().getName() + " " + "Think");
                synchronized (left) {
                    System.out.println(Thread.currentThread().getName() + " " + "left");
                    synchronized (right) {
                        System.out.println(Thread.currentThread().getName() + " " + "right");
                        Thread.sleep(1000);//进餐一段时间
                    }
                }
            }
        }
        catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }

}
```
### 解决方案
1. 服务员检查（避免策略）
2. 改变一个哲学的拿起叉子的顺序， 给叉子进行编号，每次拿起小的。
3. 餐票解决，最多n-1个餐票
4. 检测与恢复策略，发生死锁之后检测到死锁，要求一个线程释放资源。

* 检测与恢复策略：逐个终止线程，直到死锁消除；资源抢占。
终止顺序：1. 优先级，前台交互还是后台处理；2. 已占用资源，还需要的资源。3. 运行时间。
线程回退几步，但是可能导致线程饥饿。

### 有效避免的方法：
8个经验：
* 设置超时时间：`tryLock(1000,TimeUnit.MILLISECONDS)`。synchronized无法尝试锁。获取锁失败再尝试发邮件，报警等。
* 多使用并发类，不要自己设置锁。`ConcurrentHashMap`,`ConcurrentLinkedQueue`,`AtomicBoolean`
* 尽量较低锁的使用粒度和临界区。
* 如果可以使用同步代码块优先使用。
* 给线程起有意义的名字。
* 避免锁的嵌套。
* 分配资源前考虑能不能收回，银行家算法
* 不要几个功能用同一把锁。

## 活锁
持有锁之后一段时间放开锁，再重新获得锁。可能正好还是卡死，编程活锁。
活锁的特点是线程没有被阻塞，但是没有进展。活锁还要持续消耗cpu资源。双方相互谦让导致谁都无法持有锁。

* 引入随机重试，解决活锁问题。
## 饥饿
线程需要资源但是无法得到资源。
优先级的设置需要注意。
> 面试问题
1. 实际中的死锁，线程需要持有多个锁的场合，锁的嵌套。
2. 如何定位死锁，ThreadMXBean找到死锁。jstack类。
3. 解决死锁的策略。


# Java 内存模型——底层原理
## 什么是底层原理
保证Jvm在不同的cpu平台上得到的机器指令相同，因此制定了一套规范。
## JVM内存结构、Java内存模型、Java对象模型
整体方向：
* JVM内存结构，和Java虚拟机的运行时区域有关。
* Java内存模型，和Java的并发编程有关。
* Java对象模型，和Java对象在虚拟机的表现形式有关。

## JVM内存结构
* 线程共享：堆（实例对象）、方法区（永久引用等）
* 线程私有：虚拟机栈（基本数据类型，对象的引用），本地方法栈（native方法），程序计数器（字节码的行号数）。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200902203002563.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70#pic_center)

## Java 对象模型
[参考](https://blog.csdn.net/linxdcn/article/details/73287490)
更加具体，Java对象自身的存储模型
Jvm会给类创建一个instanceKlass放在方法区，用来在JVM表示该类。在java代码中new对象时，会创建一个instanceOopDesc对象，这个在栈中复制，堆中实例对象。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200902203127142.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70#pic_center)

## JMM内存模型
是一种规范，避免了依赖处理器导致结果不同，保证并发安全。在不同的虚拟机中需要遵循规则。volatile、synchronization，Lock的原理都是JMM。
三大性质：有序性 ， 可见性，原子性。
### 有序性 
Java代码的执行顺序与代码中的顺序不同。
优点：重排序之后指令优化
三种情况：
1. 编译器优化：包括JVM，编译器等
2. CPU执行重排
3. 内存的“重排序”，线程A的修改B看不到，可见性存在问题。

```java
a = 3;
b = 4;
a += 1;
// 经过重排序可能实现，先执行a = 3, 紧接着 a += 1, 最后b = 4;
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830165241397.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830165305761.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70#pic_center)

### 可见性
多线程在执行过程中，每个线程都是由自己的缓存空间`local cache`，在完成变量的修改之后，需要共享到公共内存`shared cache`才可以被其他线程看到。因此如果共享的过程较慢是会出现可见性的问题。另外一个线程无法及时感知到所读取的变量已经被另外一个线程修改了。 可以用`volatile`关键字解决。

产生可见性的原因是存在独占缓存，并不是多核。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830165334849.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70#pic_center)
#### JMM解决可见性的方法
分化为了本地内存和主内存。
1. 所有的变量都是存储在主内存中，同时每个线程也有自己独立的工作内存，工作内存中的变量内容是主内存的拷贝。
2. 线程不 能直接读写主内存中的变量，而是只能操作自己工作内存中的变量，然后再同步到主内存中。
3. 主内存是多个线程共享的，但是线程内不共享工作内存，线程的通信需要借助主内存中转完成。

## Happens-Before原则
* happens-before原则

   * 如果操作1 happens-before 操作2，那么第操作1的执行结果将对操作2可见，而且操作1的执行顺序排在第操作2之前。
	* 两个操作之间存在happens-before关系，并不意味着一定要按照happens-before原则制定的顺序来执行。**如果重排序之后的执行结果与按照happens-before关系来执行的结果一致，那么这种重排序并不非法。**


1. 单线程规则：未发生重排序，后面的语句一定能看到前面的语句发生了什么。
2. 锁操作（synchronized 和 lock）：保证锁释放以后，锁代码块内的内容是可以被看到。
3. volatile：保证修改完可以被看到。
4. 线程启动：线程启动时候，可以看到线程的run的内容。
5. 线程join原则：join对应的线程需要完全运行完，后面的语句都可以看到。
6. 传递性：
7. 中断：一个线程被其他线程interrupt时，检测中断（isInterrupt）或者抛出InterruptedException异常可以被看到。
8. 工具类的HB原则：线程安全的容器get一定能看到put的动作；CountDownLatch；Semaphore；Future；线程池，每一个任务都是能看到之间的任务结果；CyclicBarrier.

## 一个例子
这里本身是可能因为b被看到a没被看到，出现打印`a = 1, b = 3`的情况的，但是我们对b加了`volatile`关键字之后，保证了b之间的全部的操作都是可以被看到的。因此其实a一定等于3.

* 给b加了`volatile`，不进b被影响，还可以实现轻量级的同步。read代码一定看到write代码中b之前的全部操作。这是由HB原则保证的。
```java
int a;
volatile int b;
private void change(){
	a = 3;
	b = a;
}
private void print(){
	sout(b,a);
}
```
## volatile关键字
* `volatile`是一种同步机制，比synchronized和lock相关类更加轻量级，不会对上下文的切换造成很大的开销。

两点作用
* JVM可以禁止重排序，解决单例双重锁乱序
* 保证可见性：每次读取`volatile`变量之前，先使本地缓存失效，必须先到主内存中读取最新值。写`volatile`属性会立刻刷新进主内存。volatile读前插读屏障，写后加写屏障，避免CPU重排导致的问题，实现多线程之间数据的可见性。

### 使用场景
* 不适用：`a++；`无法进行原子操作的保护。
* 适合场合：`boolean flag`这种，共享变量只是被各个线程赋值，而不是被其他操作；作为刷新前的触发器。
*  `volatile`可以使`long`和`double`保持原子性。


## 原子性
* 什么是原子性
一系列的操作，要不全部执行成功，要不全部不执行，不可以执行一半。
* 原子操作有哪些
除了long和double的基本类型的赋值操作；
所有引用reference的赋值操作；
Atomic包中的原子操作。
* long和double的原子性
分为前32位读入和写入是分开的。解决的方法是用`volatile`进行修饰。
* 原子操作+原子操作！=原子操作

注意：生成对象的过程不是原子的，如`Person p = new Person()`
1. 新建一个空的Person对象；
2. 把这个对象的地址指向p；
3. 执行Person()的构造函数


## 单例模式写法，单例与并发的关系。
[参考文章](https://blog.csdn.net/zcz5566719/article/details/108369005)：
单例模式的使用理由：可以节省内存和计算，保证结果正确，方便管理。

适用场景：无状态的工具类，如日志工具类，我们只需要一个实例对象；全局信息类：记录访问次数等，没必要两个实例。

单例写法：
1. 饿汉式：静态实例和静态代码块。首先初始化了实例。最简单，但是没有延迟加载。
2. 懒汉式：在想要获取的时候才初始化实例。需要保证synchronized修饰方法。（效率低下）
3. 双重检查：同步块内部再进行一次检查。并且需要对实例加上`volatile`
	* 优点：线程安全，延迟加载，效率高
	*   为什么使用`volatile`修饰实例，利用防止重排序，保证完整的新建实例；保证可见性（其实没必要，因为synchronize的已经解决了）。
4. 静态内部类的方法。

5. 枚举的方法，最简单。

# 线程安全
无论需要怎么样的多线程访问的情况，都不需要进行额外的处理，程序都可以正确运行，就是线程安全的。
线程安全可能的代价，降低了运行速度。且难于设计。
## 线程安全问题：
* 运行结果错误
* 活跃性问题；死锁，活锁，饥饿
* 对象发布和初始化的安全问题

## 竞争条件冲突及分析

```java
package ThreadError;

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * @Author: chenzuoZhang
 * @package: ThreadError
 * @time: 2020/8/19
 * @TODO: 演示第一种线程安全错误,并且进行了升级，统计了在哪里发生错误
 */
public class CountError implements Runnable{
    static CountError instance = new CountError();
    int index = 0;
    static AtomicInteger realIndex = new AtomicInteger();
    static AtomicInteger wrongCount = new AtomicInteger();
    final boolean[] mark = new boolean[1010];
    static volatile CyclicBarrier cyclicBarrier1 = new CyclicBarrier(2);
    static volatile CyclicBarrier cyclicBarrier2 = new CyclicBarrier(2);

    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(instance);
        Thread thread2 = new Thread(instance);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println("表面结果"+instance.index);
        System.out.println("实际结果"+realIndex.get());
        System.out.println("错误结果"+wrongCount.get());
    }

    @Override
    /* 错误的写法
    public void  run(){
        for (int i = 0; i < 100; i++) {
            // 1.实际上是 a = index, a = a+1, index = a --方法：cyclicBarrier 等待大家同步
            index++;
            realIndex.incrementAndGet();
            // 2.判断可能出现问题，因为可见性，出现两个index一样--方法：mark[index] && mark[index-1]
            if (mark[index]){
                // 3. 这里可能出现bug，两个同时进入---synchronized
                wrongCount.incrementAndGet();
             }
            mark[index] = true;
        }
    }
    */
    public void run() {
        // 坑1.保证在1这个位置发生冲突也可以解决。
        mark[0] = true;
        for (int i = 0; i < 500; i++) {
            // 坑2.这里保证
            try {
                cyclicBarrier2.reset();
                cyclicBarrier1.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
            index++;
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            try {
                cyclicBarrier1.reset();
                cyclicBarrier2.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
            // 保证了两个线程同时运行到这里
            realIndex.incrementAndGet();
            synchronized (instance){
                if (mark[index] && mark[index-1]) {
                    System.out.println("发生错误" + index);
                    wrongCount.incrementAndGet();
                }
                mark[index] = true;
            }
        }
    }
}

```
## 逸出
* 方法返回了一个`private`对象
* 还没完成初始化，就把对象提供给外界。
	1. 在构造函数中未初始化完毕就this复制
	2. 隐式逸出——注册监听事件
	3. 构造函数中运行线程

解决逸出的方法：
1. 返回副本
2. 工厂模式

## 需要考虑线程安全的情况
1. 访问共享的变量或者资源，如对象的属性，静态变量，共享缓存，数据库等
2. 依赖时序的操作，及时每一步都是线程安全的，也存在并发问题：read-modify-write, check-then-act.
3. 不同数据之间存在绑定关系。
4. 使用其他类，对方没有什么是线程安全的

# 性能问题
## 调度：上下文切换
什么是上下文切换——cpu进行的操作
1. 挂起一个进程，将这个进程在CPU中的状态存储于内存中
2. 在内存中检索下一个进程的上下文，并将其在cpu的寄存器中恢复。
3. 跳转到程序计数器所指向的位置，回复进程

缓存开销：CPU重新缓存
何时导致密集上下文切换：频繁竞争锁或者IO读写

## 协作：线程调度
为了数据正确性，同步手段会禁止编译器优化，降低效率。


# 其他
* 进程和线程的区别？
进程是系统运行程序的最小单位。线程是由进程创建的，一个进程可以在执行的过程中产生多个线程。

与进程不同的是**同类的多个线程共享进程的堆和方法区资源，但每个线程有自己的程序计数器、虚拟机栈和本地方法栈**，所以系统在产生一个线程，或是在各个线程之间作切换工作时，负担要比进程小得多，也正因为如此，线程也被称为轻量级进程。

一个java程序需要启动的线程
```
public class MultiThread {
	public static void main(String[] args) {
		// 获取 Java 线程管理 MXBean
	ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
		// 不需要获取同步的 monitor 和 synchronizer 信息，仅获取线程和线程堆栈信息
		ThreadInfo[] threadInfos = threadMXBean.dumpAllThreads(false, false);
		// 遍历线程信息，仅打印线程 ID 和线程名称信息
		for (ThreadInfo threadInfo : threadInfos) {
			System.out.println("[" + threadInfo.getThreadId() + "] " + threadInfo.getThreadName());
		}
	}
}
```
	[5] Attach Listener //添加事件
	[4] Signal Dispatcher // 分发处理给 JVM 信号的线程
	[3] Finalizer //调用对象 finalize 方法的线程
	[2] Reference Handler //清除 reference 线程
	[1] main //main 线程,程序入口

* 并行和并发的区别？

这个问题在JVM的GC中就有，CMS是并发的GC，而ParNew只是并行的GC。


    并发： 同一时间段，多个任务都在执行 (单位时间内不一定同时执行)；需要来回切换
    并行： 单位时间内，多个任务同时执行。

* 什么是上下文切换？
  
一个cpu的核心只能在同一时间被一个线程使用,因此CPU需要为每个线程分配时间片并且轮转 
>当前任务在执行完 CPU 时间片切换到另一个任务之前会先保存自己的状态，以便下次再切换回这个任务时，可以再加载这个任务的状态。**任务从保存到再加载的过程就是一次上下文切换。**


![在这里插入图片描述](https://img-blog.csdnimg.cn/20210305163317186.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)